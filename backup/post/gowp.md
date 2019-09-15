---
# 常用定义
title: "golang工作池库 workerpool"           # 标题
date: 2019-09-15T11:48:31+08:00    # 创建时间
lastmod: 2019-09-15T11:48:31+08:00 # 最后修改时间
draft: false                       # 是否是草稿？
tags: ["vue", "工具", "debug"]  # 标签
categories: ["工具"]              # 分类
author: "xiexiaojun"                  # 作者

weight: 1

# 用户自定义
# 你可以选择 关闭(false) 或者 打开(true) 以下选项
comment: true   # 评论
toc: true       # 文章目录
reward: true	 # 打赏
mathjax: true    # 打开 mathjax

---
## [gowp](https://github.com/xxjwxc/gowp)
## golang worker pool ,线程池 , 工作池

- 并发限制goroutine池。
- 限制任务执行的并发性，而不是排队的任务数。
- 无论排队多少任务，都不会阻止提交任务。
- 通过队列支持

- golang 工作池公共库

### 支持最大任务数, 放到工作池里面 并等待全部完成
```
package main

import (
	"fmt"
	"time"

	"github.com/xxjwxc/gowp/workerpool"
)

func main() {
	wp := workerpool.New(10)             //设置最大线程数
	for i := 0; i < 20; i++ { //开启20个请求
		ii := i
		wp.Do(func() error {
			for j := 0; j < 10; j++ { //每次打印0-10的值
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				time.Sleep(1 * time.Second)
			}
			//time.Sleep(1 * time.Second)
			return nil
		})
	}

	wp.Wait()
	fmt.Println("down")
}
```

### 支持错误返回
```
package main

import (
	"fmt"
	"time"

	"github.com/xxjwxc/gowp/workerpool"
)

func main() {
	wp := workerpool.New(10)             //设置最大线程数
	for i := 0; i < 20; i++ { //开启20个请求
		ii := i
		wp.Do(func() error {
			for j := 0; j < 10; j++ { //每次打印0-10的值
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				if ii == 1 {
					return errors.Cause(errors.New("my test err")) //有err 立即返回
				}
				time.Sleep(1 * time.Second)
			}

			return nil
		})
	}

	err := wp.Wait()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("down")
	}
```

### 支持判断是否完成 (非阻塞)

```
package main

import (
	"fmt"
	"time"

	"github.com/xxjwxc/gowp/workerpool"
)

func main() {
	wp := workerpool.New(5)              //设置最大线程数
	for i := 0; i < 10; i++ { //开启20个请求
		//	ii := i
		wp.Do(func() error {
			for j := 0; j < 5; j++ { //每次打印0-10的值
				//fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				time.Sleep(1 * time.Second)
			}
			return nil
		})

		fmt.Println(wp.IsDone())
	}
	wp.Wait()
	fmt.Println(wp.IsDone())
	fmt.Println("down")
}
```

### 支持同步等待结果

```
package main

import (
	"fmt"
	"time"

	"github.com/xxjwxc/gowp/workerpool"
)

func main() {
	wp := workerpool.New(5)              //设置最大线程数
	for i := 0; i < 10; i++ { //开启20个请求
		ii := i
		wp.DoWait(func() error {
			for j := 0; j < 5; j++ { //每次打印0-10的值
				fmt.Println(fmt.Sprintf("%v->\t%v", ii, j))
				// if ii == 1 {
				// 	return errors.New("my test err")
				// }
				time.Sleep(1 * time.Second)
			}

			return nil
			//time.Sleep(1 * time.Second)
			//return errors.New("my test err")
		})
	}

	err := wp.Wait()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("down")
}
```

----------

## 代码地址 [gowp](https://github.com/xxjwxc/gowp)