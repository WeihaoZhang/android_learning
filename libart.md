### Heap学习

#### Heap关键类

**HeapBitmap**

为了减少指针本身所占用的内存，ART将对象的指针转换成位图里的索引，每一位指向唯一的指针。

![image-20220904093834039](pic\image-20220904093834039.png)

·HeapBitmap：它其实是一个辅助类，它内部包含continuous_space_bitmaps_和large_object_bitmaps_两个成员。这两个成员都是vector数组，所存储的元素类型分别是ContinuousSpaceBitmap和LargeObjectBitmap。·SpaceBitmap是一个模板类，模板参数表示内存地址对齐长度，取值为kObjectAlignment（值为8）表示按对象对齐，其对应的类型别名就是ContinuousSpaceBitmap。或者取值为kLargeObjectAlignment（值为4KB，一个内存页的大小）表示按页对齐，对应的类型别名为LargeObjectBitmap。

·HeapBitmap对外提供的功能也包括存储、移除、遍历对象。

#### ImageSpace相关类

Space类家族包含两个不同的派生分支——分别是Space派生分支和AllocSpace派生分支。ImageSpace是非常重要的类，它将加载编译得到的art文件.

![image-20220904101122374](pic\image-20220904101122374.png)

.art文件。它就是ART虚拟机代码里常提到的Image文件.根据art文件的来源（比如它是从哪个jar包或apk包编译得来的），Image分为boot镜像（boot image）和app镜像（app image）.·来源于某个apk的art文件称为App镜像。·来自Android系统里/system/framework下那些核心jar包的art文件统称为boot镜像。

![image-20220904102205635](pic\image-20220904102205635.png)

CMS而言，回收力度由轻到重分别是kGcTypeSticky（表示仅扫描和回收上次GC到本次GC这个时间段内所创建的对象）、kGcTypePartial（仅扫描和回收应用进程自己的内存空间，不处理zygote进程的空间。这种方式和Android中Java应用程序的创建方式有关。在Android中，应用进程是zygote进程fork出来的）、最后一种则是力度最重的类型—kGcTypeFull，它将扫描APP自己以及它从父进程zygote继承得到的堆。所以，GC回收时将依次尝试kGCTypeSticky，如果还没有空闲内存，则继续尝试kGCTypePartial，以此类推。



Heap构造函数第一部分，它主要完成了boot镜像所需art文件的加载，然后得到一系列的ImageSpace对象，最后再保存到Heap对应的成员变量中。

### JavaVMExt和JNIEnvExt

·JavaVM在JNI层中表示Java虚拟机,一个Java进程只有一个JavaVM实例.JavaVMExt类.

·JNIEnv代表JNI环境，每一个需要和Java交互的线程都有一个独立的JNIEnv对象,JNIEnvExt类.

·操作JavaVM相关接口时，其实现在java_vm_ext.cc文件的JIT类

·操作JNIEnv相关接口时，其实现在jni_internal.cc的JNI类中

### ClassLinker

在ART虚拟机的实现中，Java的某些类在虚拟机层也有对应的C++类.Object对应Java的Object类，Class对应Java的Class类。以此类推，DexCache、String、Throwable、StackTraceElement等与同名Java类相对应

#### ArtField,ArtMethod

Java源码中的class可以包含成员变量和成员函数。当class经过dex2oat编译转换后，一个类的成员变量和成员函数的信息将转换为对应的C++类，即ArtField和ArtMethod

![image-20220904165209888](pic\image-20220904165209888.png)

#### ClassTable

容器类，被ClassLoader用于管理ClassLoader所加载的类。

#### InitFromBootImage

第一步从oat文件中读取类数据。对应java中每个class实例对应的class对象。

![image-20220904172821210](pic\image-20220904172821210.png)

art文件头结构ImageHeader里的image_roots_是一个数组，该数组的第二个元素又是一个数组（ObjectArray<Class>）。它包含了37个基本类（由枚举值kJavaLangClass、kJavaLangString等标示）的类信息（由对应的mirror::Class对象描述）。这些其他的类信息则存储在ImageHeader的kSectionClassTable区域里，它包含了所有boot镜像文件里所加载的类信息

![image-20220904174422764](pic\image-20220904174422764.png)
