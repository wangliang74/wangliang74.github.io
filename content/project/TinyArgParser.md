---
title: "TinyArgParser"
date: 2026-10-16T08:00:00+08:00
draft: false
weight: 1
---

项目名称：TinyArgParser

项目地址：https://github.com/wangliang74/TinyArgParser

项目说明：
C++11 单头文件命令行参数解析器，无第三方依赖。

使用示例：
```
#include "TinyArgParser.hpp"

int main(int argc, char** argv) {
    TinyArgParser::TinyArgParser arg_parser;
    std::string filename = "default.txt";
    std::string timeout;
    bool verbose = false;

    // add_option 添加携带值的参数
    arg_parser.add_option("-f", filename, "文件路径");

    // 设置必选参数
    arg_parser.add_option("-t", timeout, "超时时间")->required();

    // add_flag 添加不携带值的参数
    arg_parser.add_flag("--version", is_verbose, "版本号");

    // 解析 -h 命令
    if (argc == 2 && argv[1] != nullptr && std::string(argv[1]) == "-h") {
        arg_parser.print_usage(argv[0]);
        return 1;
    }

    // 解析命令行参数
    if (!arg_parser.parse(argc, argv)) {
        return 1;
    }

    // 打印参数
    std::cout << "filename   : " << filename << std::endl;
    std::cout << "timeout    : " << timeout << std::endl;
    std::cout << "is_verbose : " << is_verbose << std::endl;
    return 0;
}
```

使用效果：
```
$ ./build/simple -h
Usage: ./build/simple [options]

Options:
  -f                文件路径
  -t                超时时间
  --version         版本号

$ ./build/simple -t 10 -f config.txt --version
filename   : config.txt
timeout    : 10
is_verbose : 1
```
