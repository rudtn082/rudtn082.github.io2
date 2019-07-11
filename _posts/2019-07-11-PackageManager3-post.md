---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager3"
description:
headline:
modified: 2019-07-11
category: Android
tags: [Android]
imagefeature: cover_2.jpg
mathjax:
chart:
comments: true
share: ture
featured: true
---

# [안드로이드 프레임워크 개선]


## PackageManager3


---------------------------------------

### 역할 분담  

우리는 지난주에 이어 빌드한 내용을 넥서스 5 디바이스에 플래싱 하기로 하였다.  
우여곡절 끝에 플래싱 한 디바이스는 벽돌이 되어서, 일단 역할 분담을 하고 프로젝트를 계속 진행하기로 하였다.  

1. 넥서스 5 플래싱 진행.  
2. PackageManagerService.java의 생성자 파트 코드 리뷰.  
3. 'DEBUG_PACKAGE_SCANNING' 플래그가 포함된 파트 리뷰.  

이 중에서 나는 3번 'DEBUG_PACKAGE_SCANNING' 플래그가 포함된 파트 리뷰를 맡았다.  

### 코드 분석  

Version : android-6.0.1_r77  
PackageManagerService.java  

DEBUG_PACKAGE_SCANNING 플래그는 2개의 메소드안에서 다루어지고 있다.  

#### scanDirLI  

Line : 5625 ~ 5662  

scanDirLI는 PackageManagerService에서 패키지들을 synchronized 할 때 사용된다.  

해당 메소드는 directory 내의 package를 확인한 후 package별로 scanPackageLI를 수행하며, 인스톨에 실패한 invalid package를 삭제한다.  

DEBUG_PACKAGE_SCANNING 플래그가 true일 경우에는 스캐닝 중인 app의 directory, scanFlags, parseFlags를 로그로 출력한다.  

```
private void scanDirLI(File dir, int parseFlags, int scanFlags, long currentTime) {
    final File[] files = dir.listFiles();
    if (ArrayUtils.isEmpty(files)) {
        Log.d(TAG, "No files in app dir " + dir);
        return;
    }
     //  Scanning 로그 출력  //
    if (DEBUG_PACKAGE_SCANNING) {
        Log.d(TAG, "Scanning app dir " + dir + " scanFlags=" + scanFlags
                + " flags=0x" + Integer.toHexString(parseFlags));
    }
    for (File file : files) {
        final boolean isPackage = (isApkFile(file) || file.isDirectory())
                && !PackageInstallerService.isStageName(file.getName());
        if (!isPackage) {
            // Ignore entries which are not packages
            continue;
        }
        try {
            scanPackageLI(file, parseFlags | PackageParser.PARSE_MUST_BE_APK,
                    scanFlags, currentTime, null);
        } catch (PackageManagerException e) {
            Slog.w(TAG, "Failed to parse " + file + ": " + e.getMessage());
            // Delete invalid userdata apps
            if ((parseFlags & PackageParser.PARSE_IS_SYSTEM) == 0 &&
                    e.error == PackageManager.INSTALL_FAILED_INVALID_APK) {
                logCriticalInfo(Log.WARN, "Deleting invalid package at " + file);
                if (file.isDirectory()) {
                    mInstaller.rmPackageDir(file.getAbsolutePath());
                } else {
                    file.delete();
                }
            }
        }
    }
}
```

#### scanPackageDirtyLI  

Line : 6482 ~ 7545  

scanPackageDirtyLI는 scanPackageLI에서 부르며, scanPackageLI는 scanDirLI에서 부른다.  

해당 메소드는 package를 스캔하고 paresd package를 return하며, 패키지 업데이트 여부, codePath, partition (system partition에 있는지 아니면 data partition인지), certificate, 버전 체크, 패키지가 유료인 경우, code랑 resource 가 다른 파일에 존재하지 않는 경우 수정.  

DEBUG_PACKAGE_SCANNING 플래그가 true일 경우에는 다음을 로그로 출력한다.  
** line 6545 - 스캐닝 중인 app의 name **  
** line 6572 - PackageSetting 값이 있을 때, Package의 codePath, 세팅값의 codePathString, resourcePathString **  
** line 6611 - Package의 mSharedUserId가 null이 아닌 경우, Package의 mSharedUserId, 세팅값의 userId, packages **  
** line 6939 - normal package이며 해당 directory가 존재하지 않을 경우 **  
** line 7259 - Package의 Provider가 존재할때? provider 이름, info name, Syncable한지 **  
** line 7285 - Providers의 info.name **  
** line 7305 - Services의 info.name **  
** line 7325 - receivers의 info.name **  
** line 7345 - activities의 info.name **  
** line 7379 - permissionGroups의 info.name **   
** line 7480 - Permissions의 info.name **  
** line 7508 - Instrumentation의 info.name **   

아래 코드는 내용이 많기 때문에, DEBUG_PACKAGE_SCANNING가 포함된 부분만 발췌  

```
private PackageParser.Package scanPackageDirtyLI(PackageParser.Package pkg, int parseFlags,
        int scanFlags, long currentTime, UserHandle user) throws PackageManagerException {
    //  Scanning 로그 출력  //
    if (DEBUG_PACKAGE_SCANNING) {
      if ((parseFlags & PackageParser.PARSE_CHATTY) != 0) // PackageParser
          Log.d(TAG, "Scanning package " + pkg.packageName);
    }

    :
    :

    if ((scanFlags & SCAN_REQUIRE_KNOWN) != 0) {
        if (mExpectingBetter.containsKey(pkg.packageName)) {
            logCriticalInfo(Log.WARN,
                    "Relax SCAN_REQUIRE_KNOWN requirement for package " + pkg.packageName);
        } else {
            PackageSetting known = mSettings.peekPackageLPr(pkg.packageName);
            if (known != null) {
                //  Scanning 로그 출력  //
                if (DEBUG_PACKAGE_SCANNING) {
                    Log.d(TAG, "Examining " + pkg.codePath
                            + " and requiring known paths " + known.codePathString
                            + " & " + known.resourcePathString);
                }
                if (!pkg.applicationInfo.getCodePath().equals(known.codePathString)
                        || !pkg.applicationInfo.getResourcePath().equals(known.resourcePathString)) {
                    throw new PackageManagerException(INSTALL_FAILED_PACKAGE_CHANGED,
                            "Application package " + pkg.packageName
                            + " found at " + pkg.applicationInfo.getCodePath()
                            + " but expected at " + known.codePathString + "; ignoring.");
                }
            }
        }
    }

    :
    :

    synchronized (mPackages) {
        if (pkg.mSharedUserId != null) {
            suid = mSettings.getSharedUserLPw(pkg.mSharedUserId, 0, 0, true);
            if (suid == null) {
                throw new PackageManagerException(INSTALL_FAILED_INSUFFICIENT_STORAGE,
                        "Creating application package " + pkg.packageName
                        + " for shared user failed");
            }
            //  Scanning 로그 출력  //
            if (DEBUG_PACKAGE_SCANNING) {
                if ((parseFlags & PackageParser.PARSE_CHATTY) != 0) // PackageParser
                    Log.d(TAG, "Shared UserID " + pkg.mSharedUserId + " (uid=" + suid.userId
                            + "): packages=" + suid.packages);
            }
        }

        :
        :
    }

    :
    :

    if (mPlatformPackage == pkg) {
        // The system package is special.
        dataPath = new File(Environment.getDataDirectory(), "system");
        pkg.applicationInfo.dataDir = dataPath.getPath();
    } else {
      // This is a normal package, need to make its data directory.
      dataPath = Environment.getDataUserPackageDirectory(pkg.volumeUuid,
              UserHandle.USER_OWNER, pkg.packageName);
      boolean uidError = false;
      if (dataPath.exists()) {

        :
        :

        } else {
            //  Scanning 로그 출력  //
            if (DEBUG_PACKAGE_SCANNING) {
                if ((parseFlags & PackageParser.PARSE_CHATTY) != 0) // PackageParser
                    Log.v(TAG, "Want this data dir: " + dataPath);
            }
            //invoke installer to do the actual installation
            int ret = createDataDirsLI(pkg.volumeUuid, pkgName, pkg.applicationInfo.uid,
                    pkg.applicationInfo.seinfo);
            if (ret < 0) {
                // Error from installer
                throw new PackageManagerException(INSTALL_FAILED_INSUFFICIENT_STORAGE,
                        "Unable to create data dirs [errorCode=" + ret + "]");
            }
            if (dataPath.exists()) {
                pkg.applicationInfo.dataDir = dataPath.getPath();
            } else {
                Slog.w(TAG, "Unable to create data directory: " + dataPath);
                pkg.applicationInfo.dataDir = null;
            }
        }


    :
    :

    // writer
    synchronized (mPackages) {
        // We don't expect installation to fail beyond this point

        :
        :

        int N = pkg.providers.size();
        StringBuilder r = null;
        int i;
        for (i=0; i<N; i++) {

          :
          :

          String names[] = p.info.authority.split(";");
          p.info.authority = null;
          for (int j = 0; j < names.length; j++) {

            :
            :

            if (!mProvidersByAuthority.containsKey(names[j])) {
                mProvidersByAuthority.put(names[j], p);
                if (p.info.authority == null) {
                    p.info.authority = names[j];
                } else {
                    p.info.authority = p.info.authority + ";" + names[j];
                }
                //  Scanning 로그 출력  //
                if (DEBUG_PACKAGE_SCANNING) {
                    if ((parseFlags & PackageParser.PARSE_CHATTY) != 0) // PackageParser
                        Log.d(TAG, "Registered content provider: " + names[j]
                                + ", className = " + p.info.name + ", isSyncable = "
                                + p.info.isSyncable);
                }
            }
          }
        }

        //  Scanning 로그 출력  //
        if (r != null) {
            if (DEBUG_PACKAGE_SCANNING) Log.d(TAG, "  Providers: " + r);
        }

        N = pkg.services.size();
        r = null;
        for (i=0; i<N; i++) {
            PackageParser.Service s = pkg.services.get(i); // PackageParser
            s.info.processName = fixProcessName(pkg.applicationInfo.processName,
                    s.info.processName, pkg.applicationInfo.uid);
            mServices.addService(s);
            if ((parseFlags&PackageParser.PARSE_CHATTY) != 0) {
                if (r == null) {
                    r = new StringBuilder(256);
                } else {
                    r.append(' ');
                }
                r.append(s.info.name);
            }
        }

        //  Scanning 로그 출력  //
        if (r != null) {
            if (DEBUG_PACKAGE_SCANNING) Log.d(TAG, "  Services: " + r);
        }

        :
        :

        //  이런식으로 Receivers, Activities, Permission Groups, Permissions, Instrumentation로그 출력   //

        :
        :
    }
  }
}
```

전체적인 흐름을 보았는데, 스캐닝에 관련된 내용 중 개선을 한다면 scanPackageDirtyLI 부분을 더 보거나  
PackageParser의 내용이 많이 나오는데, PackageParser.java를 정밀하게 보는 것이 좋을 것 같다.  


### 디바이스 플래싱  

![PM3_1](/images/post/PM3_1.png "PM3_1")  
