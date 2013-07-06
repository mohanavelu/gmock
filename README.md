**Google Mock** mocks generator based on libclang

### Requirements
 + [python](http://www.python.org) (tested with 2.7 and 3.2)
 + [libclang](http://clang.llvm.org) (tested with 3.2)

### Download
```
git clone --recursive git@github.com:krzysztof-jusiak/gmock.git
```

### Usage
```sh
Usage: gmock.py [options] files...

Options:
  -h, --help                show this help message and exit
  -c FILE, --config=FILE    config FILE (default='gmock.conf')
  -d DIR, --dir=DIR         dir for generated mocks (default='.')
  -l LIMIT, --limit=LIMIT   limit to interfaces within declaration (default='')
```

### Example
```sh
./gmock.py file.hpp
```
will create mocks files in current directory for all interfaces

```sh
./gmock.py -c "gmock.conf" -d "test/mocks" -l "namespace::class" file1.hpp file2.hpp
```
will create directory 'test/mocks' and mocks files within this directory for all interfaces (contains at least one pure virtual function)
which will be within 'namespace::class' declaration

### Integration with the build system
```sh
find project -iname "*.h" -or -iname "*.hpp" | xargs "project/externals/gmock.py"   \
    -c "project/conf/gmock.conf"                                                    \
    -d "project/generated/mocks"                                                    \
    -l "Project"
```

### Features
 + it's reliable (based on clang compiler)
 + it's fast (tested on project ~200 kloc -> generation of mocs takes 3-5s on common laptop)
 + output file might be easily adopted to the project via configuration file
 + easy integration with the project build system -> generate mocks files for each interface from given files limited to the project (for example via project namespace)
 + able to generate cpp files with default constructors (to speed up compilation times)
 + generate pretty output (one mock per file)
 + easy to extend (~300 lines of code)
 + handle c++ operators

```cpp
    virtual int operator()(int, double) = 0;
```

```cpp
    virtual int operator()(int arg0, double arg1) { return call_operator(arg0, arg1); }
    MOCK_METHOD2(call_operator, int(int, double));
```

### Configuration file
```python
#possible variables:
# file: interface file name
# dir: interface directory
# guard: header guard
# interface: interface class
# mock_methods: generated gmock methods
# generated_dir: generated directory
# mock_file_hpp: mock header file 
# mock_file_cpp: mock source file 

mock_file_hpp = "%(interface)sMock.hpp"

file_template_hpp = """\
/*
 * file generated by gmock: %(mock_file_hpp)s
 */
#ifndef %(guard)s
#define %(guard)s

#include <gmock/gmock.h>
#include "%(dir)s/%(file)s"

%(namespaces_begin)s

class %(interface)sMock : public %(interface)s
{
public:
%(mock_methods)s
};

%(namespaces_end)s

#endif // %(guard)s

"""

mock_file_cpp = ""
file_template_cpp = ""

```

### Example of generated output
```cpp
/*
 * file generated by gmock: I2Mock.hpp
 */
#ifndef I2MOCK_HPP
#define I2MOCK_HPP

#include <gmock/gmock.h>
#include "I2.hpp"

namespace n1 {

class I2Mock : public I2
{
public:
    MOCK_CONST_METHOD0(f0, void());
    MOCK_METHOD1(f1, void(int));
    MOCK_METHOD1(f2, void(double));
    MOCK_METHOD2(f3, void(int, double));
    MOCK_METHOD3(f4, void(int, double, const std::string &));
    MOCK_METHOD1(f5, int(const std::string &));
    MOCK_CONST_METHOD1(f6, boost::shared_ptr<int>(const boost::shared_ptr<int> &));
    MOCK_CONST_METHOD0(f7, const int&());
    MOCK_METHOD0(f8, boost::function<void(int)>());
    MOCK_CONST_METHOD1(f9, boost::non_type<int,0>(const boost::non_type<int, 1> &));
    MOCK_METHOD0(f10, const int*const ());
    MOCK_METHOD0(f11, const void());
    virtual int operator()() { return function_call_or_cast_operator(); }
    MOCK_METHOD0(function_call_or_cast_operator, int());
};

} // namespace n1

#endif // I2MOCK_HPP

```

