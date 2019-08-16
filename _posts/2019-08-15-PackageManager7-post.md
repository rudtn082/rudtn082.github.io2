---
layout: post
title: "[안드로이드 프레임워크 개선] - PackageManager7"
description:
headline:
modified: 2019-08-15
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


## PackageManager7   


---------------------------------------

시간이 오래걸리는 파트를 분석했고, 이번주는 시간이 오래 걸리는 파트에서 부르는 메소드를 분석하기로 했다.  

Version : android-6.0.1_r77  
PackageManagerService.java  

소스 코드 : https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-6.0.1_r77/services/core/java/com/android/server/pm/PackageManagerService.java  

### derivePackageAbi()  

Line : 7554 ~ 7685  

해당 메소드는 ABI를 결정하고 내부 앱인지를 판단하여 lib 라이브러리를 설치한다.  
scanPackageDirtyLI에서 부르는 메소드로, 앱의 개수마다 불리게 되어 개선이 필요해 보인다.  

① setNativeLibraryPaths(pkg)를 호출하여 lib 디렉토리를 작성  
(시스템에 내장 APK가있는 경우 system/lib, vendor/lib와 같은 시스템 APK 디렉토리에 lib 디렉토리를 작성하고)  
(사용자가 APK를 설치하는 경우 data/app-lib 아래의 pk 디렉토리와 같은 lib 디렉토리를 작성.)  

②-1 lib 라이브러리를 추출해야하는 경우, 즉 extractLibs가 true 인 경우 NativeLibraryHelper.copyNativeBinariesForSupportedAbi()를 호출하여 ABI를 일치시키고 ABI 디렉토리에 해당하는 lib 파일을 해당 디렉토리로 복사한다.  

②-2 lib 라이브러리가 필요하지 않은 경우, NativeLibraryHelper.findSupportedAbi()가 ABI와 일치하도록 호출된다.  

**새로 설치된 APK의 경우 lib 라이브러리를 추출해야 한다. 즉 extractLibs가 true.**  

③ APKprimaryCpuAbi, secondaryCpuAbi를 설정한다.  

*ABI란?*  
*API보다 저수준(바이너리에서 호환) / 두 개의 바이너리 프로그램 모듈 사이의 interface 이다. 보통 한 쪽은 라이브러리 혹은 운영체제이고, 다른 한 쪽은 사용자가 동작시키는 프로그램이다.*  

derivePackageAbi() 메소드는 다음과 같은 시간이 걸리며, 설치된 앱 개수에 따라서 오랜 시간이 걸릴 것으로 예상된다.  

![PM7_1](/images/post/PM7_1.png "PM7_1")  


```
public void derivePackageAbi(PackageParser.Package pkg, File scanFile,
                             String cpuAbiOverride, boolean extractLibs)
        throws PackageManagerException {

        setNativeLibraryPaths(pkg); // ① 호출하여 lib 디렉토리를 작성  

        if (pkg.isForwardLocked() || pkg.applicationInfo.isExternalAsec() ||
                (isSystemApp(pkg) && !pkg.isUpdatedSystemApp())) {
            extractLibs = false; // extractLibs 값 결정
        }

        final String nativeLibraryRootStr = pkg.applicationInfo.nativeLibraryRootDir;
        final boolean useIsaSpecificSubdirs = pkg.applicationInfo.nativeLibraryRootRequiresIsa;
        NativeLibraryHelper.Handle handle = null;
        try {
            handle = NativeLibraryHelper.Handle.create(scanFile);

            final File nativeLibraryRoot = new File(nativeLibraryRootStr);

            pkg.applicationInfo.primaryCpuAbi = null; // ③ 설정해야 할 것
            pkg.applicationInfo.secondaryCpuAbi = null; // ③ 설정해야 할 것

            if (isMultiArch(pkg.applicationInfo)) {

                if (pkg.cpuAbiOverride != null
                        && !NativeLibraryHelper.CLEAR_ABI_OVERRIDE.equals(pkg.cpuAbiOverride)) {
                    Slog.w(TAG, "Ignoring abiOverride for multi arch application.");
                }

                int abi32 = PackageManager.NO_NATIVE_LIBRARIES;
                int abi64 = PackageManager.NO_NATIVE_LIBRARIES;

                if (Build.SUPPORTED_32_BIT_ABIS.length > 0) {
                    if (extractLibs) { // ②-1, extractLibs가 true 인 경우
                        abi32 = NativeLibraryHelper.copyNativeBinariesForSupportedAbi(handle,
                                nativeLibraryRoot, Build.SUPPORTED_32_BIT_ABIS,
                                useIsaSpecificSubdirs);
                    } else { ②-2, extractLibs가 false 인 경우
                        abi32 = NativeLibraryHelper.findSupportedAbi(handle, Build.SUPPORTED_32_BIT_ABIS);
                    }
                }

                maybeThrowExceptionForMultiArchCopy(
                        "Error unpackaging 32 bit native libs for multiarch app.", abi32);

                if (Build.SUPPORTED_64_BIT_ABIS.length > 0) {
                    if (extractLibs) { // ②-1, extractLibs가 true 인 경우
                        abi64 = NativeLibraryHelper.copyNativeBinariesForSupportedAbi(handle,
                                nativeLibraryRoot, Build.SUPPORTED_64_BIT_ABIS,
                                useIsaSpecificSubdirs);
                    } else { ②-2, extractLibs가 false 인 경우
                        abi64 = NativeLibraryHelper.findSupportedAbi(handle, Build.SUPPORTED_64_BIT_ABIS);
                    }
                }

                maybeThrowExceptionForMultiArchCopy(
                        "Error unpackaging 64 bit native libs for multiarch app.", abi64);

                if (abi64 >= 0) {
                    pkg.applicationInfo.primaryCpuAbi = Build.SUPPORTED_64_BIT_ABIS[abi64]; // ③ ABI설정
                }

                if (abi32 >= 0) {
                    final String abi = Build.SUPPORTED_32_BIT_ABIS[abi32];
                    if (abi64 >= 0) {
                        pkg.applicationInfo.secondaryCpuAbi = abi; // ③ ABI설정
                    } else {
                        pkg.applicationInfo.primaryCpuAbi = abi; // ③ ABI설정
                    }
                }
            } else {
                String[] abiList = (cpuAbiOverride != null) ?
                        new String[] { cpuAbiOverride } : Build.SUPPORTED_ABIS;

                boolean needsRenderScriptOverride = false;
                if (Build.SUPPORTED_64_BIT_ABIS.length > 0 && cpuAbiOverride == null &&
                        NativeLibraryHelper.hasRenderscriptBitcode(handle)) {
                    abiList = Build.SUPPORTED_32_BIT_ABIS;
                    needsRenderScriptOverride = true;
                }

                final int copyRet;
                if (extractLibs) { // ②-1, extractLibs가 true 인 경우
                    copyRet = NativeLibraryHelper.copyNativeBinariesForSupportedAbi(handle,
                            nativeLibraryRoot, abiList, useIsaSpecificSubdirs);
                } else { ②-2, extractLibs가 false 인 경우
                    copyRet = NativeLibraryHelper.findSupportedAbi(handle, abiList);
                }
                if (copyRet < 0 && copyRet != PackageManager.NO_NATIVE_LIBRARIES) {
                    throw new PackageManagerException(INSTALL_FAILED_INTERNAL_ERROR,
                            "Error unpackaging native libs for app, errorCode=" + copyRet);
                }
                if (copyRet >= 0) {
                    pkg.applicationInfo.primaryCpuAbi = abiList[copyRet]; // ③ ABI설정
                } else if (copyRet == PackageManager.NO_NATIVE_LIBRARIES && cpuAbiOverride != null) {
                    pkg.applicationInfo.primaryCpuAbi = cpuAbiOverride; // ③ ABI설정
                } else if (needsRenderScriptOverride) {
                    pkg.applicationInfo.primaryCpuAbi = abiList[0]; // ③ ABI설정
                }
            }
        } catch (IOException ioe) {
            Slog.e(TAG, "Unable to get canonical file " + ioe.toString());
        } finally {
            IoUtils.closeQuietly(handle);
        }

        setNativeLibraryPaths(pkg);
    }

```



### mPackageDexOptimizer.performDexOpt()  

Line : 7081 ~ 7083  
