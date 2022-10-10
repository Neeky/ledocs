## 背景
安装了一套 hadoop 在执行 checknative 时发现有如下报错. 
```
bash bin/hadoop checknative 
2022-10-10 12:22:17,099 INFO bzip2.Bzip2Factory: Successfully loaded & initialized native-bzip2 library system-native
2022-10-10 12:22:17,102 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
2022-10-10 12:22:17,106 WARN zstd.ZStandardCompressor: Error loading zstandard native libraries: java.lang.InternalError: Cannot load libzstd.so.1 (libzstd.so.1: cannot open shared object file: No such file or directory)!
2022-10-10 12:22:17,107 WARN erasurecode.ErasureCodeNative: Loading ISA-L failed: Failed to load libisal.so.2 (libisal.so.2: cannot open shared object file: No such file or directory)
2022-10-10 12:22:17,107 WARN erasurecode.ErasureCodeNative: ISA-L support is not available in your platform... using builtin-java codec where applicable
2022-10-10 12:22:17,137 INFO nativeio.NativeIO: The native code was built without PMDK support.
Native library checking:
hadoop:  true /usr/local/hadoop-3.3.4/lib/native/libhadoop.so.1.0.0
zlib:    true /lib64/libz.so.1
zstd  :  false 
bzip2:   true /lib64/libbz2.so.1
openssl: false EVP_CIPHER_CTX_reset
ISA-L:   false Loading ISA-L failed: Failed to load libisal.so.2 (libisal.so.2: cannot open shared object file: No such file or directory)
PMDK:    false The native code was built without PMDK support.
```

---

## 分析与解决
从报错可以看出是缺少了依赖的共享库文件，这个只要找到依赖包安装上就行了。
```bash
yum -y install libzstd isa-l
```
检查
```
bash bin/hadoop checknative 
2022-10-10 12:27:02,096 INFO bzip2.Bzip2Factory: Successfully loaded & initialized native-bzip2 library system-native
2022-10-10 12:27:02,098 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
2022-10-10 12:27:02,131 INFO nativeio.NativeIO: The native code was built without PMDK support.
Native library checking:
hadoop:  true /usr/local/hadoop-3.3.4/lib/native/libhadoop.so.1.0.0
zlib:    true /lib64/libz.so.1
zstd  :  true /lib64/libzstd.so.1
bzip2:   true /lib64/libbz2.so.1
openssl: false EVP_CIPHER_CTX_reset
ISA-L:   true /lib64/libisal.so.2
PMDK:    false The native code was built without PMDK support.
```
---