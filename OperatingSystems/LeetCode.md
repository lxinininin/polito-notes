# 1116. 打印零与奇偶数

现有函数 `printNumber` 可以用一个整数参数调用，并输出该整数到控制台。

- 例如，调用 `printNumber(7)` 将会输出 `7` 到控制台。

给你类 `ZeroEvenOdd` 的一个实例，该类中有三个函数：`zero`、`even` 和 `odd` 。`ZeroEvenOdd` 的相同实例将会传递给三个不同线程：

- **线程 A：**调用 `zero()` ，只输出 `0`
- **线程 B：**调用 `even()` ，只输出偶数
- **线程 C：**调用 `odd()` ，只输出奇数

修改给出的类，以输出序列 `"010203040506..."` ，其中序列的长度必须为 `2n` 。

实现 `ZeroEvenOdd` 类：

- `ZeroEvenOdd(int n)` 用数字 `n` 初始化对象，表示需要输出的数。
- `void zero(printNumber)` 调用 `printNumber` 以输出一个 0 。
- `void even(printNumber)` 调用`printNumber` 以输出偶数。
- `void odd(printNumber)` 调用 `printNumber` 以输出奇数。

 

**示例 1：**

```
输入：n = 2
输出："0102"
解释：三条线程异步执行，其中一个调用 zero()，另一个线程调用 even()，最后一个线程调用odd()。正确的输出为 "0102"。
```

**示例 2：**

```
输入：n = 5
输出："0102030405"
```

 

**提示：**

- `1 <= n <= 1000`



```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>

int n;
sem_t sem_zero, sem_odd, sem_even;

// 打印奇数的线程
void *tOdd() {
    for (int i = 1; i <= n; i += 2) {
        sem_wait(&sem_odd);
        fprintf(stdout, "%d", i);
        sem_post(&sem_zero);
    }

    pthread_exit(NULL);
}

// 打印偶数的线程
void *tEven() {
    for (int i = 2; i <= n; i += 2) {
        sem_wait(&sem_even);
        fprintf(stdout, "%d", i);
        sem_post(&sem_zero);
    }

    pthread_exit(NULL);
}

// 打印零的主线程
int main(int argc, char* argv[]) {
    pthread_t tid1, tid2;

    if (argc != 2) {
        fprintf(stderr, "Error command\n");
        exit(EXIT_FAILURE);
    }

    n = atoi(argv[1]);

    // 初始化信号量
    sem_init(&sem_zero, 0, 1);
    sem_init(&sem_odd, 0, 0);
    sem_init(&sem_even, 0, 0);

    // 创建线程
    pthread_create(&tid1, NULL, tOdd, NULL);
    pthread_create(&tid2, NULL, tEven, NULL);

    // 主线程打印零
    for (int i = 1; i <= n; i++) {
        sem_wait(&sem_zero);
        fprintf(stdout, "%d", 0);
        if (i % 2 == 1) {
            sem_post(&sem_odd);
        } else {
            sem_post(&sem_even);
        }
    }

    // 等待子线程完成才能销毁信号量!!
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    // 销毁信号量
    sem_destroy(&sem_zero);
    sem_destroy(&sem_odd);
    sem_destroy(&sem_even);

    fprintf(stdout, "\nProgram end\n");
    exit(EXIT_SUCCESS);
}
```



# 1117. H2O 生成

现在有两种线程，氧 `oxygen` 和氢 `hydrogen`，你的目标是组织这两种线程来产生水分子。

存在一个屏障（barrier）使得每个线程必须等候直到一个完整水分子能够被产生出来。

氢和氧线程会被分别给予 `releaseHydrogen` 和 `releaseOxygen` 方法来允许它们突破屏障。

这些线程应该三三成组突破屏障并能立即组合产生一个水分子。

你必须保证产生一个水分子所需线程的结合必须发生在下一个水分子产生之前。

换句话说:

- 如果一个氧线程到达屏障时没有氢线程到达，它必须等候直到两个氢线程到达。
- 如果一个氢线程到达屏障时没有其它线程到达，它必须等候直到一个氧线程和另一个氢线程到达。

书写满足这些限制条件的氢、氧线程同步代码。

 

**示例 1:**

```
输入: water = "HOH"
输出: "HHO"
解释: "HOH" 和 "OHH" 依然都是有效解。
```

**示例 2:**

```
输入: water = "OOHHHH"
输出: "HHOHHO"
解释: "HOHHHO", "OHHHHO", "HHOHOH", "HOHHOH", "OHHHOH", "HHOOHH", "HOHOHH" 和 "OHHOHH" 依然都是有效解。
```

 

**提示：**

- `3 * n == water.length`
- `1 <= n <= 20`
- `water[i] == 'O' or 'H'`
- 输入字符串 `water` 中的 'H' 总数将会是 `2 * n` 。
- 输入字符串 `water` 中的 'O' 总数将会是 `n` 。



```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>

char* str;
pthread_mutex_t mutex_H, mutex_O;

void *tH() {
    int count = 0;

    for (int i = 0; i < strlen(str); i++) {
        if (str[i] == 'H') {
            pthread_mutex_lock(&mutex_H);

            fprintf(stdout, "%c", str[i]);
            count++;
            if (count % 2 == 1) {
                // 如果只执行了一次 H 解锁 H 再执行一次
                pthread_mutex_unlock(&mutex_H);
            } else {
                // 当执行两次 H 解锁 O 的锁
                pthread_mutex_unlock(&mutex_O);
            }
        }
    }

    pthread_exit(NULL);
}

void *tO() {
    for (int i = 0; i < strlen(str); i++) {
        if (str[i] == 'O') {
            pthread_mutex_lock(&mutex_O);

            fprintf(stdout, "%c", str[i]);

            // // 执行一次 O 解锁 H 的锁
            pthread_mutex_unlock(&mutex_H);
        }
    }

    pthread_exit(NULL);
}

int main(int argc, char* argv[]) {
    pthread_t tid1, tid2;

    if (argc != 2) {
        fprintf(stderr, "Error command\n");
        exit(EXIT_FAILURE);
    }

    str = argv[1];

    pthread_mutex_init(&mutex_H, NULL);
    pthread_mutex_init(&mutex_O, NULL);

    pthread_create(&tid1, NULL, &tH, NULL);
    pthread_create(&tid2, NULL, &tO, NULL);

    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);

    pthread_mutex_destroy(&mutex_H);
    pthread_mutex_destroy(&mutex_O);
}
```

