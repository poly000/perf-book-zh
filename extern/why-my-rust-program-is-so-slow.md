我最近开始学习Rust。我听说Rust有高性能，所以我编写了一个气泡排序程序看看。
但是这个程序和C对比运行的很慢。

__Rust代码：__

```rust
use chrono::prelude::*;

extern crate chrono;

fn bubble_sort() {
    let mut arr = vec![91, 44, 16, 32, 50, 55, 13, 12, 33, 65, 69, 71, 74, 57, 48, 61, 2, 58, 38, 27, 51, 39, 81, 79, 36, 80, 95, 23, 72, 63, 3, 30, 35, 59, 18, 28, 5, 56, 96, 40, 84, 78, 42, 8, 15, 25, 10, 43, 41, 75, 45, 99, 46, 82, 68, 76, 0, 90, 7, 49, 64, 53, 62, 89, 88, 21, 14, 4, 70, 73, 29, 98, 24, 26, 92, 97, 6, 100, 47, 67, 87, 60, 22, 52, 85, 93, 19, 31, 54, 83, 77, 20, 86, 66, 11, 37, 34, 9, 1, 94, 17];
    let l = arr.len();

    for i in 0..(l - 1) {
        for j in 0..(l - i - 1) {
            if arr[j] > arr[j + 1] {
                let t = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = t;
            }
        }
    }
}

fn main() {
    let start = Local::now().timestamp_millis();
    // println!("{}", start);
    for _r in 0..10000 {
        bubble_sort();
    }
    let end = Local::now().timestamp_millis();
    println!("{}", end - start);
}
``` 

__C 代码：__

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<math.h>
#include<sys/time.h>

void bubble_sort() {
    int arr[] = {91, 44, 16, 32, 50, 55, 13, 12, 33, 65, 69, 71, 74, 57, 48, 61, 2, 58, 38, 27, 51, 39, 81, 79, 36, 80, 95, 23, 72, 63, 3, 30, 35, 59, 18, 28, 5, 56, 96, 40, 84, 78, 42, 8, 15, 25, 10, 43, 41, 75, 45, 99, 46, 82, 68, 76, 0, 90, 7, 49, 64, 53, 62, 89, 88, 21, 14, 4, 70, 73, 29, 98, 24, 26, 92, 97, 6, 100, 47, 67, 87, 60, 22, 52, 85, 93, 19, 31, 54, 83, 77, 20, 86, 66, 11, 37, 34, 9, 1, 94, 17};
    int l = sizeof(arr) / sizeof(arr[0]);
    for (int i = 0; i < l - 1; i++) {
        for (int j = 0; j < l - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int t = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = t;
            }
        }
    }
}
int main() {
    struct timeval tv;
    
    gettimeofday(&tv, NULL);
    int start = tv.tv_sec * 1000 + tv.tv_usec / 1000;
    for (int i = 0; i < 10000; i++) {
        bubble_sort();
    }
    gettimeofday(&tv, NULL);
    int end = tv.tv_sec * 1000 + tv.tv_usec / 1000;
    printf("%d\n", end - start);
}
```

Rust版本运行在`9000ms`以内，而C版本运行在`300ms`以内。我是不是搞错了什么？

drewkett 的回答：

你运行它的时候有没有使用`--release`？ 它实现rust的优化构建。默认的是调试构建，相当于C++中的`-O0`。

`cargo run --release`
