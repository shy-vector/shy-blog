---
title: 模板元编程
published: 2025-08-15
updated: 2025-08-15
description: '本文仅举 C++ 模板元编程若干实践案例，仅供个人使用．'
image: ''
tags: [C++, SFINAE, CRTP]
category: 'C++'
draft: false
---

:::important
本文不再介绍 C++ 模板的基础用法．
:::

## 模板也是「函数」

我们可以把模板也看成一种「函数」，这个「函数」的输入输出不仅限于值，还可以是类型，甚至「函数」．

:::tip[案例 1]
统计若干类型的总大小．
:::

```cpp {"模板的输入形式":4-6} ins={"无输入时返回 0":8-10} ins={"输入个数超过 1，递归求解":12-16}
#include <print>
#include <vector>


template <class ...T>
struct get_size {};


template <>
struct get_size<> { static constexpr size_t value = 0; };


template <class T, class ...U>
struct get_size<T, U...> {
  static constexpr size_t value = sizeof(T) + get_size<U...>::value;
};

int main() {
  size_t N = get_size<int[1000], std::vector<int>, std::string>::value;
  std::println("1000 * 4 + 24 + 24 = {}", N);
}
```

```txt
1000 * 4 + 24 + 24 = 4048
```

C++ 提供了能封装输出值的类型，不用我们手搓 `value`：

```cpp
template <class ...T>
struct get_size {};

template <>
struct get_size<>
  : std::integral_constant<size_t, 0> {};

template <class T, class ...U>
struct get_size<T, U...>
  : std::integral_constant<size_t, sizeof(T) + get_size<U...>::value> {};
```

有没有函数式传参的写法？

:::warning[错误的写法]
```cpp del={"std::declval 只能用于非求值语境，且类型的移动构造函数可能被删除":12-13}
constexpr size_t get_size() { return 0; }

template <class T, class ...U>
constexpr size_t get_size(T t, U ...u) {
  return sizeof(t) + get_size(u...);
}

struct TypeA {};
struct TypeB { TypeB(TypeB &&) = delete; };

int main() {

  auto ret = get_size(std::declval<TypeA>(), std::declval<TypeB>());
}
```
:::

想要保持这种函数式写法，关键是不要真的传对象，而是传类型信息．

```cpp {"if constexpr 条件编译让我们不用到处写 get_size 特化":6-11} {"Dummy 仅包装类型":15-16}
template <class T>
struct Dummy { using type = T; };

template <class ...T>
constexpr size_t get_size(Dummy<T> ...t) {

  if constexpr (sizeof...(T) == 0) {
    return 0;
  } else {
    return (sizeof(t) + ...); // 折叠表达式实现递归
  }
}

int main() {

  auto ret = get_size(Dummy<TypeA>{}, Dummy<TypeB>{});
}
```

:::note[关于 `constexpr` 函数]
若 `constexpr` 函数中存在无法在编译期求值的参数，则 `constexpr` 函数和普通函数一样在运行时求值，此时的返回值不是常量表达式．

如果想强制要求编译时求值，可以使用 C++20 的 `consteval`，运行时求值将报错．
:::

二元 fold-expression 可以规定 `sizeof...(T)` 为 `0` 时的递归终止值，写法更简洁．另外自 C++20 开始，`std::type_identity` 可以代替手写 `Dummy` 包装类．

```cpp {"用变量模板简写表达式":6-8}
template <class ...T>
consteval size_t get_size(std::type_identity<T> ...t) {
  return (sizeof(t) + ... + 0);
}


template <class T>
constexpr std::type_identity<T> tag{};

TypeA a{};
auto ret = get_size(tag<decltype(a)>, tag<TypeB>);
```

也可以使用模板显式传参的函数式写法：

```cpp
template <class ...T>
consteval size_t get_size() { return (sizeof(T) + ... + 0); }

auto ret = get_size<TypeA, TypeB>();
```

:::tip[案例 2]
实现 `get_type<T, I>`，要求：

- 如果 `T` 为 `std::tuple<>`，返回 `void`；
- 否则，如果 `T` 为 `std::tuple<T0, ..., TN>`：
  - 如果 `0 <= I && I <= N` 为 `true`，返回 `TI`；
  - 否则，返回 `void`；
- 否则，返回 `void`．
:::

```cpp
#include <tuple>

template <class T, size_t I>
struct get_type { using type = void; };

template <size_t I>
struct get_type<std::tuple<>, I> { using type = void; };

template <class T0, class ...Ts>
struct get_type<std::tuple<T0, Ts...>, 0> { using type = T0; };

template <class T0, class ...Ts, size_t I>
struct get_type<std::tuple<T0, Ts...>, I> {
  // C++20 可以省略待决名的 typename
  using type = typename get_type<std::tuple<Ts...>, I - 1>::type;
};

int main() {
  using tup = std::tuple<int, float, bool, char, void *>;
  using ret0 = get_type<tup, 0>::type; // int
  using ret1 = get_type<tup, 1>::type; // float
  using ret2 = get_type<tup, 2>::type; // bool
  using ret3 = get_type<tup, 3>::type; // char
  using ret4 = get_type<tup, 4>::type; // void *
}
```

:::note[待决名的 `typename` 何时可以省略]
自 C++20 开始，待决名的消歧义符 `typename` 大部分都可以省略，大致除了：

- 非类型模板形参的默认值表达式中，如 `template <class T, auto val = typename T::val_type{}>`，即 `template <class T, T::val_type val = typename T::val_type{}>`；
- 模板实参类型，如 `using t = Tup<typename Tmpl<Ts>::type...>;`；
- 函数形参类型，如 `void func(typename T::val_type x) {...}`；
- 在函数或者块作用域内变量/函数声明的类型，如 `void func() { typename T::val_type val; }`、`void func() { typename T::val_type gunc(); }`．
:::

函数式写法：

```cpp
template <class T, size_t I>
consteval auto get_type_impl(
  std::type_identity<T>,
  std::integral_constant<size_t, I>
) {
  return std::type_identity<void>{};
}

template <class T0, class ...Ts, size_t I>
consteval auto get_type_impl(
  std::type_identity<std::tuple<T0, Ts...>>,
  std::integral_constant<size_t, I>
) {
  if constexpr (I == 0) {
    return std::type_identity<T0>{};
  } else {
    return get_type_impl(
      std::type_identity<std::tuple<Ts...>>{},
      std::integral_constant<size_t, I - 1>{}
    );
  }
}

template <class T, size_t I>
using get_type = decltype(get_type_impl(
  std::type_identity<T>{},
  std::integral_constant<size_t, I>{}
))::type;

int main() {
  using tup = std::tuple<int, float, char>;
  using ret0 = get_type<tup, 0>; // int
  using ret1 = get_type<tup, 1>; // float
  using ret2 = get_type<tup, 2>; // char
  using ret3 = get_type<tup, 3>; // void
}
```

也可以不包装整型参数，从函数形参转移到模板形参：

```cpp
template <size_t I, class T>
consteval auto get_type(std::type_identity<T>) {
  return std::type_identity<void>{};
}

template <size_t I, class T0, class ...Ts>
consteval auto get_type(std::type_identity<std::tuple<T0, Ts...>>) {
  if constexpr (I == 0) {
    return std::type_identity<T0>{};
  } else {
    return get_type<I - 1>(std::type_identity<std::tuple<Ts...>>{});
  }
}

int main() {
  using tup = std::tuple<int, float, char>;
  using ret0 = decltype(get_type<0>(std::type_identity<tup>{}))::type; // int
  using ret1 = decltype(get_type<1>(std::type_identity<tup>{}))::type; // float
  using ret2 = decltype(get_type<2>(std::type_identity<tup>{}))::type; // char
  using ret3 = decltype(get_type<3>(std::type_identity<tup>{}))::type; // void
}
```

:::note[包索引]
C++26 [Pack Indexing](https://www.en.cppreference.com/w/cpp/language/pack_indexing.html) 可以避免递归元编程：

```cpp
template <size_t I, class ...Ts>
consteval auto get_type(std::type_identity<std::tuple<Ts...>>) {
  if constexpr (I < sizeof...(Ts)) {
    return std::type_identity<Ts...[I]>{};
  } else {
    return std::type_identity<void>{};
  }
}
```

包索引表达式 `...[]` 支持模板参数包、函数参数包、结构化绑定包、lambda 初始化捕获包．
:::

:::tip[案例 3]
现有类型 `Tup<T1, ..., TN>`，映射 `Tmpl: T -> U`．实现模板 `mapping`，要求：

- 返回 `Tup<Tmpl<T0>, ..., Tmpl<TN>>`．
:::

```cpp {"「函数」Tmpl 作为「函数」的输入":3-5} {12-13} ins={"std::variant<int[10], char[10], bool[10]>":23-24}
#include <variant>


template <template <class> class Tmpl, class Tup>
struct mapping {};

template <
  template <class> class Tmpl,
  template <class ...> class Tup,
  class ...Ts
> struct mapping <
  Tmpl,
  Tup<Ts...>
> {
  using type = Tup<typename Tmpl<Ts>::type...>;
};

template <class T>
struct copy10 { using type = T[10]; };

int main() {
  using t = std::variant<int, char, bool>;

  using result_t = mapping<copy10, t>::type;
}
```

:::tip[案例 4]
实现模板 `array_wrapper: N -> (Tmpl: T -> T[N])`．
:::

```cpp collapse={4-16} {"输出是模板 Tmpl: T -> T[N]":20-22} ins={"std::tuple<int[7], char[7][10], bool[7]>":27-28}
#include <tuple>

// mapping 实现
template <template <class> class Tmpl, class Tup>
struct mapping {};

template <
  template <class> class Tmpl,
  template <class ...> class Tup,
  class ...Ts
> struct mapping <
  Tmpl,
  Tup<Ts...>
> {
  using type = Tup<typename Tmpl<Ts>::type...>;
};

template <size_t N>
struct array_wrapper {

  template <class T>
  struct Tmpl { using type = T[N]; };
};

int main() {
  using t = std::tuple<int, char[10], bool>;

  using result_t = mapping<array_wrapper<7>::Tmpl, t>::type;
}
```

## type_traits

标准库提供了许多很有用的「函数」，不用我们手搓．

对于大多数操作，相应的 trait 有 `noexcept` 的版本，如 `std::is_invocable_v<T, Args...>` 对应于 `std::is_nothrow_invocable_v<T, Args...>`．

对于含特殊成员函数的类型，相应的 trait 有 `trivial` 的版本，如 `std::is_constructible_v<T, Args...>` 对应于 `is_trivially_constructible_v<T, Args...>`．

### 类型判断

#### 类别

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::is_fundamental_v<T>`                     | 检查 `T` 是否为基本类型 [^1] |
| `std::is_void_v<T>`                            | 检查 `T` 是否为 `void` |
| `std::is_null_pointer_v<T>`                    | 检查 `T` 是否为 `std::nullptr_t` [^2] |
| `std::is_arithmetic_v<T>`                      | 检查 `T` 是否为算术类型 [^3] |
| `std::is_integral_v<T>`                        | 检查 `T` 是否为整型 [^4] |
| `std::is_floating_point_v<T>`                  | 检查 `T` 是否为浮点类型 |
| `std::is_enum_v<T>`                            | 检查 `T` 是否为枚举类型 |
| `std::is_pointer_v<T>`                         | 检查 `T` 是否为指针类型 [^5] |
| `std::is_member_object_pointer_v<T>`           | 检查 `T` 是否为成员变量指针类型 |
| `std::is_member_function_pointer_v<T>`         | 检查 `T` 是否为成员函数指针类型 |
| `std::is_reference_v<T>`                       | 检查 `T` 是否为引用类型 |
| `std::is_lvalue_reference_v<T>`                | 检查 `T` 是否为左值引用类型 |
| `std::is_rvalue_reference_v<T>`                | 检查 `T` 是否为右值引用类型 |

[^1]: [**基本类型**](https://cppreference.cn/w/cpp/language/types) 包括可带 cv 限定的 `void`、可带 cv 限定的 `std::nullptr_t`、整型 [^13]、浮点型，与 [**复合类型**](https://cppreference.cn/w/cpp/language/type) `std::is_compound_v<T>` 相对．

[^2]: `nullptr` 是 [`std::nullptr_t`](https://en.cppreference.com/w/cpp/types/nullptr_t.html) 的实例．\
`std::nullptr_t` 并不是指针类型，即 `std::is_pointer_v<std::nullptr_t> -> false`；\
`std::nullptr_t` 可 (显式/隐式) 转换成指针类型，如 `void *`，`char const *`；\
`std::nullptr_t` 不能在模板中做类型推导 `std::nullptr_t -> T*`．

[^3]: [**算术类型**](https://cppreference.cn/w/c/language/arithmetic_types) 包括整型、浮点类型、这些类型的 cv 限定版本．

[^4]: [**整型**](https://cppreference.cn/w/cpp/language/types#Integral_types) 包括布尔类型、字符类型、整数类型．

[^5]: 只能检测普通指针、函数指针、成员指针，不能检测智能指针，需自定义 trait，如：`template <class T> struct is_smart<std::shared_ptr<T>> : std::true_type {};`

#### 属性

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::is_const_v<T>`                           | 检查 `T` 是否带有 `const` 限定 |
| `std::is_volatile_v<T>`                        | 检查 `T` 是否带有 `volatile` 限定 |
| `std::is_empty_v<T>`                           | 检查 `T` 是否为空类型 [^6] |
| `std::is_polymorphic_v<T>`                     | 检查 `T` 是否为多态类 |
| `std::is_abstract_v<T>`                        | 检查 `T` 是否为抽象类 |
| `std::is_final_v<T>`                           | 检查 `T` 是否被 `final` 修饰 |
| `std::is_trivially_copyable_v<T>`              | 检查 `T` 是否为可平凡复制类型 [^7] |
| `std::is_trivial_v<T>`                         | 检查 `T` 是否为平凡类 [^8] |
| `std::is_standard_layout_v<T>`                 | 检查 `T` 成员的内存布局是否为标准布局 [^9] |
| `std::is_aggregate_v<T>`                       | 检查 `T` 是否为聚合类型 [^10] |
| `std::is_scoped_enum_v<T>`                     | 检查 `T` 是否为作用域枚举类型 |
| `std::is_signed_v<T>`                          | 检查 `T` 是否为有符号的算术类型 |
| `std::is_unsigned_v<T>`                        | 检查 `T` 是否为无符号的算术类型 |
| `std::is_bounded_array_v<T>`                   | 检查 `T` 是否为已知大小的数组类型 |
| `std::is_unbounded_array_v<T>`                 | 检查 `T` 是否为未知大小的数组类型 |

[^6]: 空类型不含非静态成员变量、虚函数、虚基类，如 `struct { static int x; int f(){} }`．

[^7]: [**平凡类型**](https://en.cppreference.com/w/cpp/named_req/TrivialType) 包括标量类型 [^100]、平凡类、这些类型的数组、cv 限定版本．\
称一个类 [**可平凡复制**](https://en.cppreference.com/w/cpp/language/classes.html#Trivially_copyable_class)，当且仅当其 (复制/移动)(构造函数/赋值运算符)、析构函数都是默认的，且无虚函数、虚基类．\
可平凡复制类型的内存分布是连续的，因此可以通过 `std::memcpy` 按字节将对象序列化成 `unsigned char []` 或 `std::byte []`．虽然内存连续，但内存布局和 C 语言不同，成员顺序由编译器决定 (比如访问权限相同的成员变量可能重新放在一起，C 语言不认识这些访问说明符)，因此不能兼容 C 程序．\

[^8]: 一个类在可平凡复制的基础上，具有平凡的默认构造函数，这时称它为 [**平凡类**](https://en.cppreference.com/w/cpp/language/classes.html#Trivial_class)．

[^9]: [**标准布局类型**](https://en.cppreference.com/w/cpp/named_req/StandardLayoutType.html) 包括标量类型 [^100]、标准布局类、这些类型的数组、cv 限定版本．\
一个类如果无虚函数、虚基类，所有非静态数据成员都具有相同的访问说明符，在继承体系中最多只有一个类中有非静态数据成员，在继承体系中所有类的第一个非静态成员的类型与其基类不同 (在 C++ 中，空基类的地址与第一个非静态成员共享．一旦相同，由于标准规定相同类型的对象地址必须不同，势必多分配出空间以存储基类地址，内存布局发生变化)，这时类的内存布局 (成员在内存中的排列方式) 与 C 语言的结构体完全一致，也称标准布局，这个类也被称为 [**标准布局类**](https://en.cppreference.com/w/cpp/language/classes.html#Standard-layout_class)．标准布局类允许用户自定义成员函数．\
如果平凡类型使用标准内存布局，则称这个类型为 [**POD 类型**](https://en.cppreference.com/w/cpp/named_req/PODType)，与 C 语言的类型无缝衔接，相应的 trait 为 `std::is_pod_v<T>`，该 trait 在 C++20 中弃用，取而代之的是 `std::is_trivial_v<T> && std::is_standard_layout_v<T>`．

[^10]: [**聚合类型**](https://en.cppreference.com/w/cpp/language/aggregate_initialization.html) 可以是数组，也可以是一个类：这个类没有自定义构造函数，所有非静态成员都是 `public` 的，无虚函数、虚基类，继承只能是公有继承．在 C++17 之前，这个类不能有基类．聚合类型的特色便是可以聚合初始化、在结构化绑定中可分解．

[^100]: [**标量类型**](https://en.cppreference.com/w/cpp/named_req/ScalarType.html) 包括算术类型 [^3]、枚举类型、指针类型、成员指针类型、`std::nullptr_t`、这些类型的 cv 限定版本，对应的 trait 为 `std::is_scalar_v<T>`．

#### 可支持的操作

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::is_swappable_with_v<T, U>`               | 检查 `T` 和 `U` 是否可交换 [^11] |
| `std::is_swappable_v<T>`                       | 检查 `T` 之间是否可交换 [^12] |
| `std::is_constructible_v<T, Args...>`          | 检查 `Args...` 能否构造出 `T` [^13] |
| `std::is_default_constructible_v<T>`           | 检查 `T` 能否默认构造 |
| `std::is_copy_constructible_v<T>`              | 检查 `T` 能否拷贝构造 |
| `std::is_move_constructible_v<T>`              | 检查 `T` 能否移动构造 |
| `std::is_assignable_v<T, U>`                   | 检查 `U` 能否赋值给 `T` [^14] |
| `std::is_copy_assignable_v<T>`                 | 检查 `T` 能否拷贝赋值 |
| `std::is_move_assignable_v<T>`                 | 检查 `T` 能否移动赋值 |
| `std::is_destructible_v<T>`                    | 检查 `T` 能否析构 [^15] |
| `std::has_virtual_destructor_v<T>`             | 检查 `~T()` 是否为虚函数 |
| `std::is_invocable_v<T, Args...>`              | 检查能否以参数类型 `Args...` 调用 `T` [^16] |
| `std::is_invocable_r_v<Ret, T, Args...>`       | 同上，并检查返回类型是否为 `Ret` |

[^11]: 返回 `true`，当且仅当 `std::swap(std::declval<T>(), std::declval<U>())` 在不求值语境中是良构的．

[^12]: 等价于 `std::is_swappable_with_v<T&, T&>`，只要 `T` 是可引用的类型 (其实就是非 `void`)，就返回 `true`．

[^13]: 返回 `true`，当且仅当 `T obj(std::declval<Args>()...);` 在不求值语境中是良构的．

[^14]: 返回 `true`，当且仅当 `std::declval<T>() = std::declval<U>()` 在不求值语境中是良构的．

[^15]: `T` 是引用类型，或者 `std::declval<U&>().~U()` 在不求值语境中是良构的，其中 `U` 是 `T` 移除所有多维数组维度后的类型，即 `using U = std::remove_all_extents_t<T>;`，返回 `true`；对于可带 cv 限定的 `void`、函数类型、未知边界的数组，返回 `false`．

[^16]: 只要是能被 `std::invoke` 调用的类型都被称为 `invocable`，如函数、函数指针、成员函数指针、仿函数、lambda 表达式 (匿名仿函数)、`std::function` 对象、`std::bind` 对象．`std::is_invocable_v<T>` 仅检查语法上能否调用，可触发隐式转换．

#### 类型关系

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::is_same_v<T, U>`                         | 检查 `T`, `U` 是否相同 [^17] |
| `std::is_convertible_v<From, To>`              | 检查 `From` 能否隐式转换 [^18] 成 `To` [^19]|
| `std::is_base_of_v<Base, Derived>`             | 检查 `Base` 是否为 `Derived` 的基类 |
| `std::is_virtual_base_of_v<Base, Derived>` (C++26) | 检查 `Base` 是否为 `Derived` 的虚基类 |
| `std::is_layout_compatible_v<T, U>`            | 检查 `T`, `U` 在内存布局上是否兼容 |
| `std::is_pointer_interconvertible_base_of_v<B, D>` | 检查 `D *` 能否安全转成 `B *` [^20] |
| `std::is_pointer_interconvertible_with_class(M S::*mp)` | 检查 `S *` 能否安全转成 `M *` [^21] |

[^17]: cv 限定符也要完全相同．

[^18]: [**隐式转换**](https://en.cppreference.com/w/cpp/language/implicit_conversion.html) 主要发生在算术类型的标准转换、cv 限定的添加、从派生类到基类的指针/引用转换、从基类到派生类的成员 (变量/函数) 指针转换、从数组/函数到指针的类型退化、用户定义的非 `explicit` 单参构造函数、用户定义的 `explicit` 转换函数、函数实参到形参的转换、函数返回值到返回类型的转换．

[^19]: 若 `From` 和 `To` 都是可能带 cv 限定的 `void`，则返回 `true`．

[^20]: `D` 必须是标准内存布局 [^9]，才能保证从 `D *` 转来的 `B *` 仍能正常工作，否则 `D` 的首地址将存放非空基类 `B` 的信息．此外如果 `D` 和 `B` 相同，返回 `true`．

[^21]: `s.*mp` (即 `s.m`) 与 `s` 首地址重合，相当于 `S` 必须是标准内存布局 [^8]，否则首地址将存放非空基类信息．此时 `reinterpret_cast<M&>(s)` 能唯一指代 `s.m`．此外要返回 `true`，`M` 得是对象类型，`mp` 是非空指针．

### 类型查询

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::alignment_of_v<T>`                       | 查询 `T` 的对齐字节数，即 `alignof(T)` |
| `std::rank_v<T>`                               | 查询 `T` 作为多维数组时的维数 |
| `std::extent_v<T, N>`                          | 查询 `T` 作为多维数组时在第 `N` 维的元素数量 [^22] |
| `std::invoke_result_t<F, Args...>`             | 查询以参数类型 `Args...` 调用 `T` 返回的类型 |

[^22]: 若相应维度的元素数量未知，则返回 `0`．

### 类型处理

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::add_const_t<T>`                          | `T -> const T` |
| `std::add_volatile_t<T>`                       | `T -> volatile T` |
| `std::add_cv_t<T>`                             | `T -> const volatile T` |
| `std::add_lvalue_reference_t<T>`               | `T -> T &` |
| `std::add_rvalue_reference_t<T>`               | `T -> T &&` |
| `std::add_pointer_t<T>`                        | `T -> T *` |
| `std::remove_const_t<T>`                       | `const T -> T` |
| `std::remove_volatile_t<T>`                    | `volatile T -> T` |
| `std::remove_cv_t<T>`                          | `const volatile T -> T` |
| `std::remove_reference_t<T>`                   | `T &/&& -> T` |
| `std::remove_cvref_t<T>`                       | `const volatile T &/&& -> T` |
| `std::unwrap_reference_t<T>`                   | `std::reference_wrapper<T>/T -> T/T` |
| `std::unwrap_ref_decay_t<T>`                   | `std::reference_wrapper<T>/T -> T/std::decay_t<T>` |
| `std::decay_t<T>`                              | `U[N] -> U *` / `U(...) -> U(*)(...)` / `cv U& -> U` |
| `std::remove_extent_t<T[N]>`                   | `T[N] -> T` 移除一个维度 |
| `std::remove_all_extents_t<T[I]...[N]>`        | `T[I]...[N] -> T` 移除所有维度 |
| `std::remove_pointer_t<T *>`                   | `T * -> T`，非指针类型 `T -> T` |
| `std::common_reference_t<T...>`                | 转换成 `T...` 的公共类型 |
| `std::enable_if_t<bool, T>`                    | `T -> T/`，若 `false` 则代换失败，用于 SFINAE |
| `std::conditional_t<bool, T, F>`               | `T -> T/F` |
| `std::void_t<T...>`                            | `T... -> void`，用于 SFINAE |
| `std::type_identity_t<T>`                        | `T -> T`，用于非推断语境 |

### Trait 运算

| Traits                                         | Desc                                             |
|------------------------------------------------|--------------------------------------------------|
| `std::integral_constant<T, v>`                 | 把 `v` 封装进 `static constexpr T value`，包装输出 |
| `std::bool_constant<b>`                        | `std::integral_constant<bool, b>`|
| `std::true_type / std::false_type`             | `std::bool_constant<true / false>` |
| `std::conjunction_v<B...>`                     | 对 `B...` 进行逻辑合取 [^23] |
| `std::disjunction_v<B...>`                     | 对 `B...` 进行逻辑析取 [^24] |
| `std::negation_v<B>`                           | 对 `B` 进行逻辑非 [^25] |

[^23]: 若 `sizeof...(B)` 为 `0`，则 `std::conjunction<B...>` 继承 `std::true_type`，否则继承第一个 `bool(B::value)` 为 `false` 的 `B`，若 `bool(B::value) && ...` 为 `true`，继承最后一个 `B`．`B` 通常是 `std::true_type / std::false_type`．

[^24]: 若 `sizeof...(B)` 为 `0`，则 `std::disjunction<B...>` 继承 `std::false_type`，否则继承第一个 `bool(B::value)` 为 `true` 的 `B`，若 `bool(B::value) || ...` 为 `false`，继承最后一个 `B`．`B` 通常是 `std::true_type / std::false_type`．

[^25]: 继承 `std::bool_constant<!bool(B::value)>`．

## 约束类型

### 利用 SFINAE 约束类型

在模板实参推导或函数类型推导中，代换失败并不是错误，编译器仅在重载集中抛弃该特化，不会编译失败．在 C++20 之前，我们可以利用这一点对模板类型参数进行约束．

> 这里的代换失败，发生在 *immediate context* 中，即
> 
> - 模板函数模板形参中的默认实参类型/表达式
> - 模板函数形参类型
> - 模板函数返回值类型
> - 模板类模板形参中的默认实参类型/表达式
> - 模板类模板参数列表里的类型/表达式
> 
> 函数模板签名里直接写到的类型/表达式都会出现在 *immediate context*．在这个语境因代换失败发生的错误，才被称为 *SFINAE error*．如果是触发了别的模板实例化、隐式成员生成等连锁反应而间接导致的错误是 *hard error* (会导致编译错误)．
> 
> 最经典的例子就是 `<..., class = typename B<T>::type>`，要想取出 `type`，必须实例化 `B<T>`，一旦实例化失败 (比如 `B<T>` 不含 `type`)，则发生 *hard error*．

:::tip[案例 1]
对 `template <size_t N> void foo() {}` 中的 `N` 约束：

- `N` 为偶数
:::

根据 *SFINAE error* 触发的位置和方式，可以有下面几种写法：

```cpp {"在形参类型处诱发数组长度为 0 的 SFINAE error":1-5} {"std::enable_if 协助实现 SFINAE":7-11} {"在返回值类型处使用 SFINAE":13-17} {"默认模板实参处使用 SFINAE":19-23}
// 1
template <size_t N>
void foo(char (*)[N % 2 == 0] = nullptr) {
  std::println("N is even");
}

// 2
template <size_t N>
void foo(std::enable_if_t<N % 2 == 0, int> = 0) {
  std::println("N is even");
}

// 3
template <size_t N>
auto foo() -> decltype(std::declval<int[N % 2 == 0]>(), void()) {
  std::println("N is even");
}

// 4
template <size_t N, std::enable_if_t<N % 2 == 0, int> = 0>
void foo() {
  std::println("N is even");
}
```

### 利用 Concept 约束类型

## CRTP

CRTP (Curiously Recurring Template Pattern) 可以在编译期间把派生类的类型作为模板参数传递给基类．

### 实现「静态多态」/「接口约定」

所谓「静态多态」就是在编译期确定行为绑定，通过统一接口适配不同类型的具体实现．函数重载就是一种「静态多态」．

CRTP 通过模板参数将派生类型注入基类，使得基类能在编译期调用派生类的具体实现，从而实现基类定义接口、派生类提供实现的「静态多态」．

CRTP 让派生类的基类各不相同，**失去了在抽象层处理的能力**，比如将不同派生类型的对象放入基类容器里、通过基类指针调用具体派生类实现等；而动态多态就可以，并能在运行时确定具体派生类型，从而调用具体派生类的实现．

:::note[另一种角度]
上面是网上很多人使用的表述．也可以这么认为：CRTP 起到**使不同类存在若干相同接口的成员函数**的约定作用，没有什么继承关系．
:::

```cpp {"要求 T 具有 value 属性，can_level_up、fix_impl、attack_impl 方法":5-14} {"所谓的「静态多态」，某种程度上就是通过模板实现的":41-43} del={"没有在抽象层处理的能力":46-47}
template <class T>
class Weapon {
protected:
  float power;

public:
  void attack() {
    auto that = static_cast<T*>(this);
    if (that->can_level_up()) {
      that->fix_impl();
      power += that->value;
    }
    that->attack_impl();
  }

  void fix() {
    auto that = static_cast<T*>(this);
    that->fix_impl();
  }
};

class Bow : public Weapon<Bow> {
  friend class Weapon<Bow>;
protected:
  int value;
public:
  bool can_level_up() { /* ... */ }
  void fix_impl() { /* ... */ }
  void attack_impl() { /* ... */ }
};

class Gun : public Weapon<Gun> {
  friend class Weapon<Gun>;
protected:
  int value;
public:
  bool can_level_up() { /* ... */ }
  void fix_impl() { /* ... */ }
  void attack_impl() { /* ... */ }
};

template <class T>
void fix_weapon(Weapon<T> &wp) { wp.fix(); }

int main() {

  std::vector<std::unique_ptr<Weapon>> wps;
}
```

C++23 可以向非静态成员函数显式传入对象自身，在此之前都是通过隐式传入 `this` 指针实现的．

```cpp {5-9} {23-24}
struct A {
  void f(int x, int y) const;             // void f(int, int) const &;
  void g(float x, bool y);                // void g(float, bool) &;

  void func(this A &self, int x);         // void func(int) &;
  void func(this A const &self, int x);   // void func(int) const;
  void func(this A &&self, int x);        // void func(int) &&;
  void func(this A const &&self, int x);  // void func(int) const &&;
  void gunc(this A self, float y);        // 按值传入对象自身

  // void gunc(this A &self, float y);    // 会和 gunc(this A, float) 歧义
  void hunc(this A &self) {               // void hunc() &;
    // this->func(7);                     // 使用 this-deducing 后无法再使用 this
    self.func(7);                         // this->func(7) / func(7)
  }
};

A a;
A const ca;
A *pa = &a;
A const *pca = &ca;

void (A::*pf)(int, int) const = &A::f;    // 成员函数指针
void (*pfunc)(A const &, int) = &A::func; // 普通函数指针

a.f(4, 9); (a.*pf)(4, 9); (pa->*pf)(4, 9); std::invoke(pf, a, 4, 9);
// pf(a, 4, 9); pf(pa, 4, 9);             // pf 是成员函数指针，需 std::invoke 调用
ca.func(7); pfunc(ca, 7); std::invoke(pfunc, ca, 7);
// (ca.*pfunc)(7); (pca->*pfunc)(7);      // pfunc 是普通函数指针

struct B {
  template <class T>
  void f(this T& self)                    // & / const &
  template <class T>
  void func(this T&& self);               // & / const & / && / const &&
  void gunc(this auto&& self);            // 同上，转发引用
};
```

`self` 是实打实的派生对象而不是基类指针，这意味着我们不再需要把指针类型转换到实际派生类型了．

```cpp {"如果用 Weapon &，类 auto&& 将推导为 Weapon &，此时无 fix_impl":39-42} {"能用基类指针表示也没用，无法获得派生类型相当于残废":48-51}
class Weapon {
protected:
  float power;

public:
  void attack(this auto&& self) {
    if (self.can_level_up()) {
      self.fix_impl();
      self.power += self.value;
    }
    self.attack_impl();
  }

  void fix(this auto&& self) {
    self.fix_impl();
  }
};

class Bow : public Weapon {
  friend class Weapon;
protected:
  int value;
public:
  bool can_level_up() { /* ... */ }
  void fix_impl() { /* ... */ }
  void attack_impl() { /* ... */ }
};

class Gun : public Weapon {
  friend class Weapon;
protected:
  int value;
public:
  bool can_level_up() { /* ... */ }
  void fix_impl() { /* ... */ }
  void attack_impl() { /* ... */ }
};


template <class T>
  requires std::is_base_of_v<Weapon, T>
void fix_weapon(T &wp) { wp.fix(); }

int main() {
  Bow a;
  fix_weapon(a);


  std::unique_ptr<Weapon> b = std::make_unique<Gun>();
  // fix_weapon(*b);
  std::vector<std::unique_ptr<Weapon>> v;
}
```

### 实现「注入实现」

CRTP 的另一个作用就是复用代码的通用逻辑，实现自动化定义成员函数．

:::tip[案例 1]
现有基类 `Planet` 以及派生类 `Sun`，`Earth`，对这两个派生类使用 *单例模式*．
:::

```cpp
struct Planet {
  float mass, radius, heat;
  float get_volume() const {
    return 3.14f * 4 / 3 * radius * radius * radius;
  }
};

class Sun : public Planet {
private:
  Sun() { /* ... */ };
  ~Sun() { /* ... */ };
  Sun(Sun const &) = delete;
  Sun& operator=(Sun const &) = delete;

public:
  static Sun& get_instance() {
    static Sun instance = Sun();
    return instance;
  }

  void light(Planet& p) {
    float dH = heat / get_volume();
    p.heat += dH;
    heat -= dH;
  }
};

class Earth : public Planet {
private:
  Earth() { /* ... */ };
  ~Earth() { /* ... */ };
  Earth(Earth const &) = delete;
  Earth& operator=(Earth const &) = delete;

public:
  static Earth& get_instance() {
    static Earth instance = Earth();
    return instance;
  }
};
```

我们发现这种 *单例模式* 的逻辑高度通用．

:::warning[失败的实践]
尝试使用协议类的形式提取这种逻辑：

```cpp del={"get_instance() 返回的不是 Sun 类型":34-36}
class Singleton {
protected:
  Singleton() = default;
  ~Singleton() = default;
  Singleton(Singleton const &) = delete;
  Singleton& operator=(Singleton const &) = delete;

public:
  static Singleton& get_instance() {
    static Singleton instance = Singleton();
    return instance;
  }
};

class Sun : public Singleton, public Planet {
protected:
  Sun() { /* ... */ }
  ~Sun() { /* ... */ }
public:
  void light(Planet& p) {
    float dH = heat / get_volume();
    p.heat += dH;
    heat -= dH;
  }
};

class Earth : public Singleton, public Planet {
protected:
  Earth() { /* ... */ }
  ~Earth() { /* ... */ }
};

int main() {
  // 1
  Sun& s = Sun::get_instance();
  Sun&& t = std::move(Sun::get_instance());

  // Sun my_sun, another_sun;
}
```

最大的问题就是协议类的单例模式根本没有应用在派生类上：`Sun::get_instance()` 得到的是 `Singleton` 的单一实例而非 `Sun` 的单一实例．我们提取逻辑的同时，类型信息也被抹去了．
:::

我们只是想复用「禁止拷贝，通过 `get_instance()` 获取唯一实例」这种模式，并在类 `Singleton` 统一实现这种模式，但又缺乏应用该模式的类型信息．如果实现内容**仅类型不同**，这个时候就能使用 CRTP 实现「自动化实现」：

```cpp ins="Singleton<Sun>" ins={"约定 T 具有的实现":6-13} ins={"Singleton<Sun>::get_instance() 需要访问 Sun() 和 ~Sun() 以管理生命周期":17-18}
template <class T>
class Singleton {
protected:
  Singleton() = default;
  ~Singleton() = default;

  Singleton(Singleton const &) = delete;
  Singleton& operator=(Singleton const &) = delete;
public:
  static T& get_instance() {
    static T instance = T();
    return instance;
  }
};

class Sun : public Singleton<Sun>, public Planet {

  friend class Singleton<Sun>;
protected:
  Sun() { /* ... */ }
  ~Sun() { /* ... */ }
public:
  void light(Planet& p) {
    float dH = heat / get_volume();
    p.heat += dH;
    heat -= dH;
  }
};

class Earth : public Singleton<Earth>, public Planet {
  friend class Singleton<Earth>;
protected:
  Earth() { /* ... */ }
  ~Earth() { /* ... */ }
};
```

:::note[另一种视角？]
也许可以这样理解：CRTP 允许构建一个特别的「函数」，输入是类型、值、「函数」，输出是实现，并把输出 **注入** 到派生类里．
:::

:::tip[案例 2]
现有 4 个类 `Ball`、`Cube`、`Cat`、`Dog`，其中 `Ball`、`Cube` 的基类是 `Object`，`Cat`、`Dog` 的基类是 `Animal`．实现：

- `Object` 的深拷贝方法；
- `Animal` 的深拷贝方法．
:::

我们的拷贝操作发生在上层模块，不知道待拷贝对象的具体类型，如果直接调用上层拷贝构造函数，会发生 *对象切片* (object-slicing)，丢失类型信息．

假设现在有

```cpp
Ball x; Cube y; Cat z; Dog w;
```

浅拷贝返回值实际上指向的还是原来的对象，没有真正拷贝：

```cpp
struct Object {
  float object_data;
  std::unique_ptr<Object> clone() {
    return std::unique_ptr<Object>(this);
  }
};

struct Animal {
  float animal_data;
  std::unique_ptr<Animal> clone() {
    return std::unique_ptr<Animal>(this);
  }
};

struct Ball : Object { float ball_data; };
struct Cube : Object { float cube_data; };
struct Cat : Animal { float cat_data; };
struct Dog : Animal { float dog_data; };
```

而深拷贝的 `new` 操作又必须要求得知具体类型：

```cpp
std::unique_ptr<Object> ret;
ret = std::make_unique<Ball>(x);
ret = std::make_unique<Cube>(y);
ret = std::make_unique<Cat>(z);
ret = std::make_unique<Dog>(w);
```

虽然深拷贝成功，但显式写出了对象的具体类型，违反了开闭原则．

如果愿意，可以保留虚函数接口，把实现下放：

```cpp
struct Object {
  float object_data;
  virtual ~Object() = default;
  virtual std::unique_ptr<Object> clone() = 0;
};

struct Animal {
  float animal_data;
  virtual ~Animal() = default;
  virtual std::unique_ptr<Animal> clone() = 0;
};

struct Ball : Object {
  float ball_data;
  std::unique_ptr<Object> clone() override {
    return std::make_unique<Ball>(*this);
  }
};

struct Cube : Object {
  float cube_data;
  std::unique_ptr<Object> clone() override {
    return std::make_unique<Cube>(*this);
  }
};

struct Cat : Animal {
  float cat_data;
  std::unique_ptr<Animal> clone() override {
    return std::make_unique<Cat>(*this);
  }
};

struct Dog : Animal {
  float dog_data;
  std::unique_ptr<Animal> clone() override {
    return std::make_unique<Dog>(*this);
  }
};
```

超级复读机！我们可以使用 CRTP 提取这种通用的逻辑，直接在基类实现它们！

:::warning[错误的实践]

```cpp
template <class Derived>
struct Object {
  float object_data;
  std::unique_ptr<Object> clone() {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

template <class Derived>
struct Animal {
  float animal_data;
  std::unique_ptr<Animal> clone() {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

struct Ball : Object<Ball> { float ball_data; };
struct Cube : Object<Cube> { float cube_data; };
struct Cat : Animal<Cat> { float cat_data; };
struct Dog : Animal<Dog> { float dog_data; };
```

CRTP 的副作用就是让基类变成了模板，导致派生类的基类各不相同，失去了动态派发的能力：

```cpp
Ball x; Cube y;
std::shared_ptr<Object<Ball>> nx = x.clone(); // are you kidding me?
std::shared_ptr<Object<Cube>> ny = y.clone(); // 这还不是违反了开闭原则？
```
:::

要让基类 `Base` 免受 CRTP 的副作用，保留基类指针指向派生类对象的能力，我们可以试试套一层辅助类 `BaseImpl`：

:::warning[还不完全正确的实践]

```cpp
struct Object { float object_data; };

template <class Derived>
struct ObjectImpl : Object {
  std::unique_ptr<Object> clone() {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

struct Animal { float animal_data; };

template <class Derived>
struct AnimalImpl : Animal {
  std::unique_ptr<Animal> clone() {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

struct Ball : ObjectImpl<Ball> { float ball_data; };
struct Cube : ObjectImpl<Cube> { float cube_data; };
struct Cat : AnimalImpl<Cat> { float cat_data; };
struct Dog : AnimalImpl<Dog> { float dog_data; };
```

你会发现的确能够用基类指针指向派生类对象，但本质上你还是失去了多态的 `clone`：

```cpp ins={"x 成功通过 ObjectImpl<Ball>::clone() 深拷贝":2-3} del={"nx 没有 clone()！失去了多态能力":4-5}
Ball x;

std::unique_ptr<Object> nx = x.clone();

std::unique_ptr<Object> nnx = nx->clone();
```
:::

因此若想保留动态多态，基类 `Base` 的虚函数接口无论如何也不能丢，然后在 `BaseImpl` 辅助类里利用 CRTP 实现虚函数！

```cpp
struct Object {
  float object_data;
  virtual ~Object() = default;
  virtual std::unique_ptr<Object> clone() = 0;
};

template <class Derived>
struct ObjectImpl : Object {
  std::unique_ptr<Object> clone() override {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

struct Animal {
  float animal_data;
  virtual ~Animal() = default;
  virtual std::unique_ptr<Animal> clone() = 0;
};

template <class Derived>
struct AnimalImpl : Animal {
  std::unique_ptr<Animal> clone() override {
    auto that = static_cast<Derived *>(this);
    return std::make_unique<Derived>(*that);
  }
};

struct Ball : ObjectImpl<Ball> { float ball_data; };
struct Cube : ObjectImpl<Cube> { float cube_data; };
struct Cat : AnimalImpl<Cat> { float cat_data; };
struct Dog : AnimalImpl<Dog> { float dog_data; };
```

:::note
举个例子，`AnimalImpl<Cat>` 往 `Cat` 里注入了 `std::unique_ptr<Animal> clone()`，`Derived` 对应 `Cat`，减少了复读机的感觉．这个时候 CRTP 显然不是为了消除虚表开销，CRTP 并非与 `virtual` 水火不容．
:::

感觉还是有点复读机，还能再提取吗？只要想保留动态多态，通过 CRTP 注入 `clone` 实现的方式并不能 `override` 基类的 `clone`，`override` 就是无法避免的，就算再写一个 `Copyable<Base, Derived>::copy_impl` 注入，也意义不大了．

## 编译期 for 循环

<!--
TODO:
C++ 无法在一个类的偏特化中注入另外一个偏特化：tag dispatch / traits 实现偏特化“继承” 或者 if constexpr
自己来发明浮点数
std::make_index_sequence
模板实参推导
待决类名的名称查找问题
偏特化偏序关系
__cdecl,__fastcall,__stdcall,x64
非推导语境
initializer_list
copy elision

// 1. traits 定义
template <typename T>
struct dist_traits {
    using result_type = long double;
    static result_type call(T x1, T y1, T x2, T y2) {
        // 整型专用实现
        return std::sqrt(
            static_cast<long double>(x1 - x2) * (x1 - x2) +
            static_cast<long double>(y1 - y2) * (y1 - y2)
        );
    }
};
template <typename T>
struct dist_traits<T, std::enable_if_t<std::is_floating_point_v<T>>> {
    using result_type = T;
    static result_type call(T x1, T y1, T x2, T y2) {
        // 浮点专用实现
        return std::sqrt((x1 - x2)*(x1 - x2) + (y1 - y2)*(y1 - y2));
    }
};

// C++17
template <typename T>
struct Dot2 : Vec<T, 2> {
    T x, y;
    // ...
    friend auto dist(Dot2 const& u, Dot2 const& v) {
        return dist_traits<T>::call(u.x, u.y, v.x, v.y);
    }
};

template <typename T>
struct Dot2 : Vec<T, 2> {
    T x, y;
    friend auto dist(Dot2 const& u, Dot2 const& v) {
        if constexpr (std::is_floating_point_v<T>) {
            return std::sqrt((u.x - v.x)*(u.x - v.x) + (u.y - v.y)*(u.y - v.y));
        } else {
            return std::sqrt(
                static_cast<long double>(u.x - v.x) * (u.x - v.x) +
                static_cast<long double>(u.y - v.y) * (u.y - v.y)
            );
        }
    }
};
-->