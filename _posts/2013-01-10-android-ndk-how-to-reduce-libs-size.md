---
layout: post
title: 'Android NDK: How to Reduce Binaries Size - The Algolia Blog'
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

When we started Algolia Development for Android, binary size optimization was
not one of our main concerns. In fact we even started to develop in JAVA
before switching to C/C++ for [reasons of performance][1].

We were reminded of the importance of binary size by [Cyril
Mottier][2] who informed us that it would be
difficult to integrate our lib in [AVelov][3] Android Application because its
size. AVelov is 638KB and Algolia was 850KB, which would mean that AVelov
would more than double in size with Algolia Search embedded.

To address this problem we managed to reduce Algolia binary size from 850KB to
307KB. In this post we share how we did it.

### Do not use Exceptions and RTTI

We actually do not use exceptions in our native lib, but for the sake of
completeness, I'll cover this point too.

C++ exceptions and RTTI are disabled by default but you can enable them via
_APP_CPPFLAGS_ in your _Application.mk_ file and use a compatible STL, for
example:

    
    APP_CPPFLAGS += -fexceptions -frtti
    APP_STL := stlport_shared

Whilst using exceptions and RTTI can help you to use existing code, it will
obviously increase your binary size. If you have a way to remove them, go for
it! Actually, there's another reason to avoid using C++ exceptions: their
support is still far from perfect. For example if was impossible for us to
catch a C++ exception and launch a Java exception in JNI. The following code
results in a crash (will probably be fixed in a future release of the Android
NDK toolchain):

    
    try {
        ...
    } catch (std::exception& e) {
        env->ThrowNew(env->FindClass("java/lang/Exception"), "Error occured");
    }

### Do not use iostream

When starting to investigate our library size following Cyril's feedback, we
discovered that Algolia binaries had vastly increased in size since our last
release (from 850KB to 1.35MB)! We first suspected the Android NDK toolchain
since we upgraded it and tested different toolchains, but we only observed
minor changes.

By dichotomy search in our commits, we discovered that a single line of code
was responsible for the inflation:

    
    std::cerr << .... << std::endl;

As incredible as it may sound, using iostream increases a lot the binary size.
Our tests shown that it adds a least 300KB per architecture! You must be very
careful with iostream and prefer to use __android_log_print method:

    
    #include <android/log.h>
    #define APPNAME "MyApp"
    
    __android_log_print(ANDROID_LOG_VERBOSE, APPNAME, "The value of 1 + 1 is %d", 1+1);

Make sure you also link against the logging library, in your Android.mk file:

    
    LOCAL_LDLIBS := -llog

### Use -fvisibility=hidden

An efficient way to reduce binary size is to use the [visibility
feature][4] of gcc. This feature lets you
control which functions will be exported in the symbols table. Hopefully, JNI
comes with a _JNIEXPORT_ macro that flags JNI functions as public. You just
have to check that all functions used by JNI are prefixed by JNIEXPORT, like
this one:

    
    JNIEXPORT void JNICALL Java_ClassName_MethodName
    (JNIEnv *env, jobject obj, jstring javaString)

Then you have just to add _-fvisibility=hidden_ for C and C++ files in
_Android.mk_ file:

    
    LOCAL_CPPFLAGS += -fvisibility=hidden
    LOCAL_CFLAGS += -fvisibility=hidden

In our case the binaries were down to 809KB (-5%) but remember the gains may
be very different for your project. Make your own measures!

### Discard Unused Functions with gc-sections

Another interesting approach is to remove unused code in the binary. It can
drastically reduce its size if for example part of your code is only used for
tests.

To enable this feature, you just have to change the C and C++ compilation
flags and the linker flags in _Android.mk_:

    
    LOCAL_CPPFLAGS += -ffunction-sections -fdata-sections
    LOCAL_CFLAGS += -ffunction-sections -fdata-sections 
    LOCAL_LDFLAGS += -Wl,--gc-sections

Of course you can combine this feature with the visibility one:

    
    LOCAL_CPPFLAGS += -ffunction-sections -fdata-sections -fvisibility=hidden
    LOCAL_CPPFLAGS += -ffunction-sections -fdata-sections -fvisibility=hidden
    LOCAL_CFLAGS += -ffunction-sections -fdata-sections  LOCAL_LDFLAGS += -Wl,--gc-sections

This optim only got us a 1% gain, but once combined with the previous
visibility one, we were down to 691KB (-18.7%).

### Remove Duplicated Code

You can remove duplicated code with the --icf=safe option of the linker. Be
careful, this option will probably remove your code inlining, you must check
that this flag does not impact performance.

This option is not yet available on the mips architecture so you need to add
an architecture check in _Android.mk_:

    
    ifneq ($(TARGET_ARCH),mips)
      LOCAL_LDFLAGS += -Wl,--icf=safe
    endif

And if you want to combine this option with gc-sections:

    
    ifeq ($(TARGET_ARCH),mips)
      LOCAL_LDFLAGS += -Wl,--gc-sections
    else
      LOCAL_LDFLAGS += -Wl,--gc-sections,--icf=safe
    endif

We actually only obtained a 0.8% gain in size with this one. All previous
optimizations combined, we were now at 687KB (-19.2%).

### Change the Default Flags of the Toolchain

If you want to go even further, you can change the default compilation flags
of the toolchain. Flags are not identical accross architectures, for example:

  * inline-limit is set to 64 for arm and set to 300 for x86 and mips
  * Optimization flag is set to -Os (optimize for size) for arm and set to -O2 (optimize for performance) for x86 and mips

As arm is used by the large majority of devices, we have applied arm settings
for other architectures. Here is the patch we applied on the toolchain
(version r8d):

    
    --- android-ndk-r8d/toolchains/mipsel-linux-android-4.6/setup.mk
    +++ android-ndk-r8d.new/toolchains/mipsel-linux-android-4.6/setup.mk
    @@ -41,12 +41,12 @@
     TARGET_C_INCLUDES := 
         $(SYSROOT)/usr/include
    
    -TARGET_mips_release_CFLAGS := -O2 
    +TARGET_mips_release_CFLAGS := -Os 
                                   -g 
                                   -DNDEBUG 
                                   -fomit-frame-pointer 
                                   -funswitch-loops     
    -                              -finline-limit=300
    +                              -finline-limit=64
    
     TARGET_mips_debug_CFLAGS := -O0 
                                 -g 
    --- android-ndk-r8d/toolchains/x86-4.6/setup.mk
    +++ android-ndk-r8d.new/toolchains/x86-4.6/setup.mk
    @@ -39,13 +39,13 @@
    
     TARGET_CFLAGS += -fstack-protector
    
    -TARGET_x86_release_CFLAGS := -O2 
    +TARGET_x86_release_CFLAGS := -Os 
                                  -g 
                                  -DNDEBUG 
                                  -fomit-frame-pointer 
                                  -fstrict-aliasing    
                                  -funswitch-loops     
    -                             -finline-limit=300
    +                             -finline-limit=64
    
     # When building for debug, compile everything as x86.
     TARGET_x86_debug_CFLAGS := $(TARGET_x86_release_CFLAGS)

We were good for a 8.5% gain with these new flags. Once combined with previous
optimizations, we were now at 613KB (-27.9%).

### Limit the Number of Architectures

Our final suggestion is to limit the number of architectures. Supporting
armeabi-v7a is mandory for performance if you have a lot of floating point
computation, but armeabi will provide a similar result if you do not need a
FPU. As for mips processors... well they just are not in use on the market
today.

And if binary size is really important to you, you can just limit your support
to armeabi and x86 architectures in _Application.mk_:

    
    APP_ABI := armeabi x86

Obviously, this optim was the killer one. Dropping two out of four
architectures halved the binaries size. Overall we obtained a size of 307KB, a
64% gain from the initial 850KB (not counting the bump at 1.35MB due to
iostream).

### Conclusion

I hope this post will help you to reduce the size of your native libraries on
Android since default flags are far from optimal. Don't expect to obtain the
same size reductions, they will highly depend on your specific usage. And if
you know other methods to reduce binary size, please share in the comments!


[1]: http://blog.algolia.com/need-performance-on-mobile-use-c-cpp/
[2]: http://android.cyrilmottier.com
[3]: https://play.google.com/store/apps/details?id=com.cyrilmottier.android.avelov
[4]: http://gcc.gnu.org/wiki/Visibility
