# CMake

你或许听过好几种 Make 工具，例如 GNU Make，Qt 的 qmake，微软的 MS nmake，BSD Make（pmake），Makepp 等等。这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。这样就带来了一个严峻的问题：如果软件想跨平台，必须保证能够在不同平台的编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile，这将是一件令人抓狂的工作。

CMake就是针对上面问题所设计的工具：它首先允许开发者编写一种平台无关的 CMakeLists.txt 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile和工程文件，如 Unix 的Makefile 或 Windows 的 Visual Studio工程。从而做到“一次编写，多处运行”。显然，CMake是一个比上述几种 make 更高级的编译配置工具一些使用 CMake 作为项目架构系统的知名开源项目有 VTK、ITK、KDE、OpenCV、OSG 等。

