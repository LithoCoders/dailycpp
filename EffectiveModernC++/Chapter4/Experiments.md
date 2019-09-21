This post does not belong to any item in the book, but I think it is a good idea to try out some code here and not pollute other items :-)
# What is 'undefined behavior' ?
The following code declares two pointers, which point to the same memory location. After deallocating memory location pointed by `a`, we can not free memory pointed by `b` any more. If we try, this is *undefined behavior*. `b` is called *dangling* pointer.
```c++
#include <iostream>

int main()
{
    int i = 1;
    int* a = new int(i);
    int* b = a;
    std::cout << *a << std::endl;
    *b += 1;
    std::cout << *a << std::endl;
    delete a;
    delete b;
    
    return 0;
}

Start

1
2

*** Error in `./prog.exe': double free or corruption (fasttop): 0x0000000000ce1150 ***
======= Backtrace: =========
/lib/x86_64-linux-gnu/libc.so.6(+0x777e5)[0x7f77b37c57e5]
/lib/x86_64-linux-gnu/libc.so.6(+0x8037a)[0x7f77b37ce37a]
/lib/x86_64-linux-gnu/libc.so.6(cfree+0x4c)[0x7f77b37d253c]
./prog.exe[0x400dcb]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7f77b376e830]
./prog.exe[0x400c89]
======= Memory map: ========
00400000-00402000 r-xp 00000000 fd:03 9586150                            /home/jail/prog.exe
00601000-00602000 rw-p 00001000 fd:03 9586150                            /home/jail/prog.exe
00ccd000-00cff000 rw-p 00000000 00:00 0                                  [heap]
7f77ac000000-7f77ac021000 rw-p 00000000 00:00 0 
7f77ac021000-7f77b0000000 ---p 00000000 00:00 0 
7f77b2f18000-7f77b2f1b000 r-xp 00000000 fd:03 147110                     /lib/x86_64-linux-gnu/libdl-2.23.so
7f77b2f1b000-7f77b311a000 ---p 00003000 fd:03 147110                     /lib/x86_64-linux-gnu/libdl-2.23.so
7f77b311a000-7f77b311b000 r--p 00002000 fd:03 147110                     /lib/x86_64-linux-gnu/libdl-2.23.so
7f77b311b000-7f77b311c000 rw-p 00003000 fd:03 147110                     /lib/x86_64-linux-gnu/libdl-2.23.so
7f77b311c000-7f77b312b000 r-xp 00000000 fd:03 132308                     /lib/x86_64-linux-gnu/libbz2.so.1.0.4
7f77b312b000-7f77b332a000 ---p 0000f000 fd:03 132308                     /lib/x86_64-linux-gnu/libbz2.so.1.0.4
7f77b332a000-7f77b332b000 r--p 0000e000 fd:03 132308                     /lib/x86_64-linux-gnu/libbz2.so.1.0.4
7f77b332b000-7f77b332c000 rw-p 0000f000 fd:03 132308                     /lib/x86_64-linux-gnu/libbz2.so.1.0.4
7f77b332c000-7f77b3345000 r-xp 00000000 fd:03 131208                     /lib/x86_64-linux-gnu/libz.so.1.2.8
7f77b3345000-7f77b3544000 ---p 00019000 fd:03 131208                     /lib/x86_64-linux-gnu/libz.so.1.2.8
7f77b3544000-7f77b3545000 r--p 00018000 fd:03 131208                     /lib/x86_64-linux-gnu/libz.so.1.2.8
7f77b3545000-7f77b3546000 rw-p 00019000 fd:03 131208                     /lib/x86_64-linux-gnu/libz.so.1.2.8
7f77b3546000-7f77b354d000 r-xp 00000000 fd:03 147108                     /lib/x86_64-linux-gnu/librt-2.23.so
7f77b354d000-7f77b374c000 ---p 00007000 fd:03 147108                     /lib/x86_64-linux-gnu/librt-2.23.so
7f77b374c000-7f77b374d000 r--p 00006000 fd:03 147108                     /lib/x86_64-linux-gnu/librt-2.23.so
7f77b374d000-7f77b374e000 rw-p 00007000 fd:03 147108                     /lib/x86_64-linux-gnu/librt-2.23.so
7f77b374e000-7f77b390e000 r-xp 00000000 fd:03 147125                     /lib/x86_64-linux-gnu/libc-2.23.so
7f77b390e000-7f77b3b0e000 ---p 001c0000 fd:03 147125                     /lib/x86_64-linux-gnu/libc-2.23.so
7f77b3b0e000-7f77b3b12000 r--p 001c0000 fd:03 147125                     /lib/x86_64-linux-gnu/libc-2.23.so
7f77b3b12000-7f77b3b14000 rw-p 001c4000 fd:03 147125                     /lib/x86_64-linux-gnu/libc-2.23.so
7f77b3b14000-7f77b3b18000 rw-p 00000000 00:00 0 
7f77b3b18000-7f77b3b2f000 r-xp 00000000 fd:03 1851694                    /opt/wandbox/gcc-head/lib64/libgcc_s.so.1
7f77b3b2f000-7f77b3d2e000 ---p 00017000 fd:03 1851694                    /opt/wandbox/gcc-head/lib64/libgcc_s.so.1
7f77b3d2e000-7f77b3d2f000 rw-p 00016000 fd:03 1851694                    /opt/wandbox/gcc-head/lib64/libgcc_s.so.1
7f77b3d2f000-7f77b3e37000 r-xp 00000000 fd:03 147128                     /lib/x86_64-linux-gnu/libm-2.23.so
7f77b3e37000-7f77b4036000 ---p 00108000 fd:03 147128                     /lib/x86_64-linux-gnu/libm-2.23.so
7f77b4036000-7f77b4037000 r--p 00107000 fd:03 147128                     /lib/x86_64-linux-gnu/libm-2.23.so
7f77b4037000-7f77b4038000 rw-p 00108000 fd:03 147128                     /lib/x86_64-linux-gnu/libm-2.23.so
7f77b4038000-7f77b4201000 r-xp 00000000 fd:03 1848461                    /opt/wandbox/gcc-head/lib64/libstdc++.so.6.0.28
7f77b4201000-7f77b4401000 ---p 001c9000 fd:03 1848461                    /opt/wandbox/gcc-head/lib64/libstdc++.so.6.0.28
7f77b4401000-7f77b440c000 r--p 001c9000 fd:03 1848461                    /opt/wandbox/gcc-head/lib64/libstdc++.so.6.0.28
7f77b440c000-7f77b440f000 rw-p 001d4000 fd:03 1848461                    /opt/wandbox/gcc-head/lib64/libstdc++.so.6.0.28
7f77b440f000-7f77b4412000 rw-p 00000000 00:00 0 
7f77b4412000-7f77b443c000 r-xp 00000000 fd:03 437363                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wserialization.so.1.70.0
7f77b443c000-7f77b463c000 ---p 0002a000 fd:03 437363                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wserialization.so.1.70.0
7f77b463c000-7f77b463e000 rw-p 0002a000 fd:03 437363                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wserialization.so.1.70.0
7f77b463e000-7f77b4708000 r-xp 00000000 fd:03 437309                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wave.so.1.70.0
7f77b4708000-7f77b4907000 ---p 000ca000 fd:03 437309                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wave.so.1.70.0
7f77b4907000-7f77b490c000 rw-p 000c9000 fd:03 437309                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_wave.so.1.70.0
7f77b490c000-7f77b491e000 r-xp 00000000 fd:03 437308                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_type_erasure.so.1.70.0
7f77b491e000-7f77b4b1d000 ---p 00012000 fd:03 437308                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_type_erasure.so.1.70.0
7f77b4b1d000-7f77b4b1f000 rw-p 00011000 fd:03 437308                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_type_erasure.so.1.70.0
7f77b4b1f000-7f77b4b26000 r-xp 00000000 fd:03 437307                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_timer.so.1.70.0
7f77b4b26000-7f77b4d25000 ---p 00007000 fd:03 437307                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_timer.so.1.70.0
7f77b4d25000-7f77b4d26000 rw-p 00006000 fd:03 437307                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_timer.so.1.70.0
7f77b4d26000-7f77b4d4c000 r-xp 00000000 fd:03 437306                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_thread.so.1.70.0
7f77b4d4c000-7f77b4f4b000 ---p 00026000 fd:03 437306                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_thread.so.1.70.0
7f77b4f4b000-7f77b4f4e000 rw-p 00025000 fd:03 437306                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_thread.so.1.70.0
7f77b4f4e000-7f77b4f4f000 r-xp 00000000 fd:03 437305                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_system.so.1.70.0
7f77b4f4f000-7f77b514e000 ---p 00001000 fd:03 437305                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_system.so.1.70.0
7f77b514e000-7f77b514f000 rw-p 00000000 fd:03 437305                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_system.so.1.70.0
7f77b514f000-7f77b5150000 r-xp 00000000 fd:03 437304                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_noop.so.1.70.0
7f77b5150000-7f77b534f000 ---p 00001000 fd:03 437304                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_noop.so.1.70.0
7f77b534f000-7f77b5350000 rw-p 00000000 fd:03 437304                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_noop.so.1.70.0
7f77b5350000-7f77b5353000 r-xp 00000000 fd:03 437303                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_basic.so.1.70.0
7f77b5353000-7f77b5552000 ---p 00003000 fd:03 437303                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_basic.so.1.70.0
7f77b5552000-7f77b5553000 rw-p 00002000 fd:03 437303                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_basic.so.1.70.0
7f77b5553000-7f77b5557000 r-xp 00000000 fd:03 437302                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_addr2line.so.1.70.0
7f77b5557000-7f77b5757000 ---p 00004000 fd:03 437302                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_addr2line.so.1.70.0
7f77b5757000-7f77b5758000 rw-p 00004000 fd:03 437302                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_stacktrace_addr2line.so.1.70.0
7f77b5758000-7f77b5797000 r-xp 00000000 fd:03 437301                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_serialization.so.1.70.0
7f77b5797000-7f77b5997000 ---p 0003f000 fd:03 437301                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_serialization.so.1.70.0
7f77b5997000-7f77b599a000 rw-p 0003f000 fd:03 437301                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_serialization.so.1.70.0
7f77b599a000-7f77b5a6c000 r-xp 00000000 fd:03 437300                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_regex.so.1.70.0
7f77b5a6c000-7f77b5c6c000 ---p 000d2000 fd:03 437300                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_regex.so.1.70.0
7f77b5c6c000-7f77b5c71000 rw-p 000d2000 fd:03 437300                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_regex.so.1.70.0
7f77b5c71000-7f77b5c7a000 r-xp 00000000 fd:03 437299                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_random.so.1.70.0
7f77b5c7a000-7f77b5e7a000 ---p 00009000 fd:03 437299                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_random.so.1.70.0
7f77b5e7a000-7f77b5e7b000 rw-p 00009000 fd:03 437299                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_random.so.1.70.0
7f77b5e7b000-7f77b5f00000 r-xp 00000000 fd:03 437298                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_program_options.so.1.70.0
7f77b5f00000-7f77b6100000 ---p 00085000 fd:03 437298                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_program_options.so.1.70.0
7f77b6100000-7f77b6105000 rw-p 00085000 fd:03 437298                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_program_options.so.1.70.0
7f77b6105000-7f77b6154000 r-xp 00000000 fd:03 437297                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1l.so.1.70.0
7f77b6154000-7f77b6354000 ---p 0004f000 fd:03 437297                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1l.so.1.70.0
7f77b6354000-7f77b6355000 rw-p 0004f000 fd:03 437297                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1l.so.1.70.0
7f77b6355000-7f77b635a000 rw-p 00000000 00:00 0 
7f77b635a000-7f77b63a5000 r-xp 00000000 fd:03 437296                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1f.so.1.70.0
7f77b63a5000-7f77b65a4000 ---p 0004b000 fd:03 437296                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1f.so.1.70.0
7f77b65a4000-7f77b65a6000 rw-p 0004a000 fd:03 437296                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1f.so.1.70.0
7f77b65a6000-7f77b65a9000 rw-p 00000000 00:00 0 
7f77b65a9000-7f77b65fb000 r-xp 00000000 fd:03 437295                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1.so.1.70.0
7f77b65fb000-7f77b67fa000 ---p 00052000 fd:03 437295                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1.so.1.70.0
7f77b67fa000-7f77b67fc000 rw-p 00051000 fd:03 437295                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_tr1.so.1.70.0
7f77b67fc000-7f77b6800000 rw-p 00000000 00:00 0 
7f77b6800000-7f77b6814000 r-xp 00000000 fd:03 437362                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99l.so.1.70.0
7f77b6814000-7f77b6a13000 ---p 00014000 fd:03 437362                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99l.so.1.70.0
7f77b6a13000-7f77b6a14000 rw-p 00013000 fd:03 437362                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99l.so.1.70.0
7f77b6a14000-7f77b6a15000 rw-p 00000000 00:00 0 
7f77b6a15000-7f77b6a28000 r-xp 00000000 fd:03 437293                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99f.so.1.70.0
7f77b6a28000-7f77b6c28000 ---p 00013000 fd:03 437293                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99f.so.1.70.0
7f77b6c28000-7f77b6c29000 rw-p 00013000 fd:03 437293                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99f.so.1.70.0
7f77b6c29000-7f77b6c3d000 r-xp 00000000 fd:03 437292                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99.so.1.70.0
7f77b6c3d000-7f77b6e3d000 ---p 00014000 fd:03 437292                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99.so.1.70.0
7f77b6e3d000-7f77b6e3e000 rw-p 00014000 fd:03 437292                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_math_c99.so.1.70.0
7f77b6e3e000-7f77b6e3f000 rw-p 00000000 00:00 0 
7f77b6e3f000-7f77b6eeb000 r-xp 00000000 fd:03 437290                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log_setup.so.1.70.0
7f77b6eeb000-7f77b70eb000 ---p 000ac000 fd:03 437290                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log_setup.so.1.70.0
7f77b70eb000-7f77b70f1000 rw-p 000ac000 fd:03 437290                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log_setup.so.1.70.0
7f77b70f1000-7f77b70f3000 rw-p 00000000 00:00 0 
7f77b70f3000-7f77b71e0000 r-xp 00000000 fd:03 437289                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log.so.1.70.0
7f77b71e0000-7f77b73e0000 ---p 000ed000 fd:03 437289                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log.so.1.70.0
7f77b73e0000-7f77b73ea000 rw-p 000ed000 fd:03 437289                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_log.so.1.70.0
7f77b73ea000-7f77b7483000 r-xp 00000000 fd:03 437288                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_locale.so.1.70.0
7f77b7483000-7f77b7682000 ---p 00099000 fd:03 437288                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_locale.so.1.70.0
7f77b7682000-7f77b7688000 rw-p 00098000 fd:03 437288                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_locale.so.1.70.0
7f77b7688000-7f77b76a2000 r-xp 00000000 fd:03 437287                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_iostreams.so.1.70.0
7f77b76a2000-7f77b78a1000 ---p 0001a000 fd:03 437287                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_iostreams.so.1.70.0
7f77b78a1000-7f77b78a3000 rw-p 00019000 fd:03 437287                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_iostreams.so.1.70.0
7f77b78a3000-7f77b78e9000 r-xp 00000000 fd:03 437285                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_graph.so.1.70.0
7f77b78e9000-7f77b7ae8000 ---p 00046000 fd:03 437285                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_graph.so.1.70.0
7f77b7ae8000-7f77b7aeb000 rw-p 00045000 fd:03 437285                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_graph.so.1.70.0
7f77b7aeb000-7f77b7b06000 r-xp 00000000 fd:03 437284                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_filesystem.so.1.70.0
7f77b7b06000-7f77b7d06000 ---p 0001b000 fd:03 437284                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_filesystem.so.1.70.0
7f77b7d06000-7f77b7d07000 rw-p 0001b000 fd:03 437284                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_filesystem.so.1.70.0
7f77b7d07000-7f77b7d18000 r-xp 00000000 fd:03 437283                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_fiber.so.1.70.0
7f77b7d18000-7f77b7f18000 ---p 00011000 fd:03 437283                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_fiber.so.1.70.0
7f77b7f18000-7f77b7f19000 rw-p 00011000 fd:03 437283                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_fiber.so.1.70.0
7f77b7f19000-7f77b7f28000 r-xp 00000000 fd:03 437361                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_date_time.so.1.70.0
7f77b7f28000-7f77b8127000 ---p 0000f000 fd:03 437361                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_date_time.so.1.70.0
7f77b8127000-7f77b8129000 rw-p 0000e000 fd:03 437361                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_date_time.so.1.70.0
7f77b8129000-7f77b8136000 r-xp 00000000 fd:03 462263                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_coroutine.so.1.70.0
7f77b8136000-7f77b8335000 ---p 0000d000 fd:03 462263                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_coroutine.so.1.70.0
7f77b8335000-7f77b8336000 rw-p 0000c000 fd:03 462263                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_coroutine.so.1.70.0
7f77b8336000-7f77b8356000 r-xp 00000000 fd:03 465542                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_contract.so.1.70.0
7f77b8356000-7f77b8556000 ---p 00020000 fd:03 465542                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_contract.so.1.70.0
7f77b8556000-7f77b8558000 rw-p 00020000 fd:03 465542                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_contract.so.1.70.0
7f77b8558000-7f77b855a000 r-xp 00000000 fd:03 477056                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_context.so.1.70.0
7f77b855a000-7f77b875a000 ---p 00002000 fd:03 477056                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_context.so.1.70.0
7f77b875a000-7f77b875b000 rw-p 00002000 fd:03 477056                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_context.so.1.70.0
7f77b875b000-7f77b876e000 r-xp 00000000 fd:03 462260                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_container.so.1.70.0
7f77b876e000-7f77b896e000 ---p 00013000 fd:03 462260                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_container.so.1.70.0
7f77b896e000-7f77b896f000 rw-p 00013000 fd:03 462260                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_container.so.1.70.0
7f77b896f000-7f77b8979000 r-xp 00000000 fd:03 503637                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_chrono.so.1.70.0
7f77b8979000-7f77b8b78000 ---p 0000a000 fd:03 503637                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_chrono.so.1.70.0
7f77b8b78000-7f77b8b79000 rw-p 00009000 fd:03 503637                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_chrono.so.1.70.0
7f77b8b79000-7f77b8b7a000 r-xp 00000000 fd:03 520347                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_atomic.so.1.70.0
7f77b8b7a000-7f77b8d7a000 ---p 00001000 fd:03 520347                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_atomic.so.1.70.0
7f77b8d7a000-7f77b8d7b000 rw-p 00001000 fd:03 520347                     /opt/wandbox/boost-1.70.0/gcc-head/lib/libboost_atomic.so.1.70.0
7f77b8d7b000-7f77b8d93000 r-xp 00000000 fd:03 147112                     /lib/x86_64-linux-gnu/libpthread-2.23.so
7f77b8d93000-7f77b8f92000 ---p 00018000 fd:03 147112                     /lib/x86_64-linux-gnu/libpthread-2.23.so
7f77b8f92000-7f77b8f93000 r--p 00017000 fd:03 147112                     /lib/x86_64-linux-gnu/libpthread-2.23.so
7f77b8f93000-7f77b8f94000 rw-p 00018000 fd:03 147112                     /lib/x86_64-linux-gnu/libpthread-2.23.so
7f77b8f94000-7f77b8f98000 rw-p 00000000 00:00 0 
7f77b8f98000-7f77b8fbe000 r-xp 00000000 fd:03 147111                     /lib/x86_64-linux-gnu/ld-2.23.so
7f77b919c000-7f77b91b5000 rw-p 00000000 00:00 0 
7f77b91bc000-7f77b91bd000 rw-p 00000000 00:00 0 
7f77b91bd000-7f77b91be000 r--p 00025000 fd:03 147111                     /lib/x86_64-linux-gnu/ld-2.23.so
7f77b91be000-7f77b91bf000 rw-p 00026000 fd:03 147111                     /lib/x86_64-linux-gnu/ld-2.23.so
7f77b91bf000-7f77b91c0000 rw-p 00000000 00:00 0 
7fff23bae000-7fff23bcf000 rw-p 00000000 00:00 0                          [stack]
7fff23be9000-7fff23bec000 r--p 00000000 00:00 0                          [vvar]
7fff23bec000-7fff23bee000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]

Aborted

Finish
```
