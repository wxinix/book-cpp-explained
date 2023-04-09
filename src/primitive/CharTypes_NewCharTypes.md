# New Character Types

## Why `char` not good for UTF-8
In C++, `char` is a fundamental type that represents a byte-sized unit of data. Historically, it has been used to represent both ASCII characters and other narrow character sets, depending on the execution environment. However, the use of char for representing Unicode text can be problematic, as the encoding and interpretation of Unicode text can vary between platforms and locales.

Suppose we have the following UTF-8 string:

```cpp
const char* utf8str = u8"你吃饭了吗?";
std::cout << utf8str << std::endl;
```

We might expect to see the output "你吃饭了吗?" in the console. However, if the execution character set of the platform is not set to UTF-8, the output may appear garbled or incorrect.

>The "execution character set of the platform" refers to the character encoding scheme used by the operating system and/or the compiler to represent text data internally in a computer program.

>In C and C++, the execution character set determines how characters are represented in the char data type. The specific character set used can vary depending on the platform, compiler, and locale settings.

>For example, on Windows systems, the default execution character set is typically based on the Windows-1252 code page, which is a superset of ASCII that includes characters for European languages. On Unix-based systems, the default execution character set is typically based on the ASCII encoding.

## UTF-related character types

`char8_t` was introduced in C++20 to provide a distinct type that is guaranteed to represent an 8-bit code unit of UTF-8 encoded Unicode text. This allows for safer and more efficient handling of UTF-8 strings, as developers can use char8_t to represent individual code units of the UTF-8 encoding. This can help to avoid issues such as misinterpreting multi-byte sequences or incorrectly handling invalid code points. For example, in the following code, the output will be correct regardless of the execution character set of the platform.

```cpp
const char8_t* utf8str = u8"你吃饭了吗?";
std::cout << utf8str << std::endl;
```

`char16_t` and `char32_t` were introduced in C++11 to provide support for Unicode text encoding. `char16_t` represents a 16-bit code unit of UTF-16 encoded Unicode text, while `char32_t` represents a 32-bit code unit of UTF-32 encoded Unicode text. 

| Type        | Introduced in | Main Reason for Introduction | Literal Prefix | Sample Code                                      |
|-------------|---------------|------------------------------|----------------|-------------------------------------------------|
| `char8_t`      | C++20         | UTF-8 encoding | `u8`         | `const char8_t* str = u8"吃了吗";`       |
| `char16_t`     | C++11         | UTF-16 encoding | `u`          | `const char16_t* str = u"吃了吗";`       |
| `char32_t`     | C++11         | UTF-32 encoding | `U`          | `const char32_t* str = U"吃了吗";`       |

The string literal prefix `u8`, `u`, `U` were introduced in C++11. The following code won't pass compilation with C++11 because they cannot be applied to characters. It is since C++17 that these literal prefix are allowed to be used with a character.

```cpp
char utf8c = u8'a'; // C++11 will fail but C++17/20 can pass
```

Also the following code would fail compiling because the value cannot fit a single byte.
```cpp
char utf8c = u8'好';
```

## Print UTF-8 string to console

`std::cout` cannot be used to output UTF-8 string to console. Use `printf` instead.

```cpp
#include <iostream>

using namespace std;

int main() {
  char8_t utf8Chars[] = u8"你好世界"; // Null terminator automatically appended.
  char8_t utf8CharsWithNullTerminator[] = u8"你好世界\0"; // Will have two null terminators.

  auto len_1 = std::char_traits<char8_t>::length(utf8Chars);
  auto len_2 = std::char_traits<char8_t>::length(utf8CharsWithNullTerminator);

  cout << "length(utf8Chars) = " 
       << len_1 
       << endl; // output 12

  cout << "length(utf8CharsWithNullTerminator) = " 
       << len_2 
       << endl; // output 12

  cout << "sizeof(char8_t) = " 
       << sizeof(char8_t) 
       << endl; // output 1
  
  // std::cout << utf8Words << std::endl; // This would fail compiling.  
  printf("%s", reinterpret_cast<char*>(&utf8Chars[0]));

  /*
  for (std::size_t i = 0; i < len; i++) {
    std::cout << utf8Chars[i] << '\n'; // This would fail compiling.
  }
  */

  return 0;
}

```
