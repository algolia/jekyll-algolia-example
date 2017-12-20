---
layout: post
title: Why Android APK Format is a Mistake
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

When I started to develop for Android it appeared to me that an APK file was
just an archive, a simple approach that you can find in many systems today.
Files are extracted from the archive at installation and you can access them
via the file-system.

This seemed even more reasonable since Android uses Linux which is very good
in respect to POSIX standards.

But I was completely wrong! An APK is not a mere archive: the application
starts from and uses the APK at runtime! This is a horrible decision that will
probably hurt Android for a long time...

**[Edit 28-Jan-2013]** The goal of this post was to express my point of view about the bad properties of using directly the APK file at runtime versus relying on the file system. I used memory-mapped file to illustrate this but the post is incorrect on that topic. There is in fact a way to memory-map a file directly from the APK: you can use an extension for which files are stored uncompressed inside the APK (mp3, jpg, ...) and use the `AssetManager.openFD()` or `Resources.openRawResourceFD()` to have offset/length inside the APK file.

All my thanks to Jay Freeman for his excellent feedback. His comments helped
me to understand my mistake and to improve our Android integration!

**[/Edit]**  

### What is the Problem with the APK format?

Let's look at our own example. At Algolia, we have designed an efficient
binary data-structure that is able to answer instant-search queries in just a
few milliseconds, even with a very big data set (for example with all the
world cities' names from Geonames: 3M+ entries). This data-structure is
designed to be used directly on disk without being copied in memory. To obtain
optimal performance, we use a [memory-mapped
file][1] which is standard on
all platforms, especially on Linux.

We have been able to use memory-mapped files on all platforms, except on
Android!  In fact you can only retrieve an InputStream from a file packaged in
an APK. So the only solution to use a memory-mapped file is to copy the
resource from the APK on disk and then to use the file-system. This seems like
re-implementing an installer in each application.

### Is the APK so bad? Why did they design it this way?

I imagine that Android developers chose this approach to solve some pitfalls
of file-systems. I can think for example about solving performance problems
when you have a lot of small files in one folder, or reducing the size of
applications on the device (resources are compressed in the APK and
decompressed only when the application uses them, which actually contributes
to the sluggish image of Android).

I may of course be wrong, there may be other more important reasons for this
approach. But if not, Android should have thought more about the consequences
of their choice: in the long term, the APK constraints are more serious than
those small pitfalls that could have been solved in other ways.

But wait... Android applications can contain dynamic libraries (.so files) via
NDK. Isn't it the principle of dynamic libraries to be memory-mapped? In fact
I am pretty sure they discovered this problem when working on NDK since
dynamic libraries are automatically extracted from APK file at installation
and stored in an application directory in '/data/data'. I am wondering why
they decided to implement this hack instead of fixing the problem...

### Conclusion

Developing an API, a SDK or worse, a whole platform, is extremely difficult.
Let's face it, it's unavoidable to ship some badly designed components or
inconsistent APIs. We definitely need to listen to developers' feedback even
when it hurts. Actually, the real difficulty comes when it's time to put
things right without alienating existing users!

By the way, if you know more about APK design choices, I'm interested to hear
from you!


[1]: http://en.wikipedia.org/wiki/Memory-mapped_file
