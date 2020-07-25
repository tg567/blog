---
title: Golang易遗忘知识点
tags: Golang基础
categories: Golang
---

个人笔记
## 1.len cap append关系
``` code
func main() {
	a1 := make([]int, 5)
	log.Printf("a1 len:%d, cap:%d\n\n", len(a1), cap(a1))

	b1 := []int{1, 2}
	b2 := append(b1, 3)
	b3 := append(b1, 3)
	b4 := append(b1, 3, 4, 5, 6)
	b5 := append(b1, 3, 4, 5, 6, 7, 8, 9)
	b6 := make([]int, 1024)
	b7 := append(b6, 1)
	log.Printf("b1 len:%d, cap:%d, addr:%p\n", len(b1), cap(b1), b1)
	log.Printf("b2 len:%d, cap:%d, addr:%p\n", len(b2), cap(b2), b2)
	log.Printf("b3 len:%d, cap:%d, addr:%p\n", len(b3), cap(b3), b3)
	log.Printf("b4 len:%d, cap:%d, addr:%p\n", len(b4), cap(b4), b4)
	log.Printf("b5 len:%d, cap:%d, addr:%p\n", len(b5), cap(b5), b5)
	log.Printf("b7 len:%d, cap:%d\n\n", len(b7), cap(b7))

	c1 := []int{1, 2, 3, 4}
	c2 := append(c1, 5)
	c3 := append(c2, 6)
	c4 := append(c2, 7, 8)
	log.Printf("c1 len:%d, cap:%d, addr:%p, value:%v\n", len(c1), cap(c1), c1, c1)
	log.Printf("c2 len:%d, cap:%d, addr:%p, value:%v\n", len(c2), cap(c2), c2, c2)
	log.Printf("c3 len:%d, cap:%d, addr:%p, value:%v\n", len(c3), cap(c3), c3, c3)
	log.Printf("c4 len:%d, cap:%d, addr:%p, value:%v\n\n", len(c4), cap(c4), c4, c4)

	d1 := c3[:8]
	log.Printf("d1 len:%d, cap:%d, addr:%p, value%v\n", len(d1), cap(d1), d1, d1)
	d2 := d1[1:]
	log.Printf("d2 len:%d, cap:%d, addr:%p, value%v\n\n", len(d2), cap(d2), d2, d2)
}
```
输出:
``` output
2020/07/25 16:08:03 a1 len:5, cap:5

2020/07/25 16:08:03 b1 len:2, cap:2, addr:0xc000012800
2020/07/25 16:08:03 b2 len:3, cap:4, addr:0xc00010eb40
2020/07/25 16:08:03 b3 len:3, cap:4, addr:0xc00010eb60
2020/07/25 16:08:03 b4 len:6, cap:6, addr:0xc000110630
2020/07/25 16:08:03 b5 len:9, cap:10, addr:0xc000112280
2020/07/25 16:08:03 b7 len:1025, cap:1280

2020/07/25 16:08:03 c1 len:4, cap:4, addr:0xc00010eba0, value:[1 2 3 4]
2020/07/25 16:08:03 c2 len:5, cap:8, addr:0xc00000e7c0, value:[1 2 3 4 5]
2020/07/25 16:08:03 c3 len:6, cap:8, addr:0xc00000e7c0, value:[1 2 3 4 5 7]
2020/07/25 16:08:03 c4 len:7, cap:8, addr:0xc00000e7c0, value:[1 2 3 4 5 7 8]

2020/07/25 16:08:03 d1 len:8, cap:8, addr:0xc00000e7c0, value[1 2 3 4 5 7 8 0]
2020/07/25 16:08:03 d2 len:7, cap:7, addr:0xc00000e7c8, value[2 3 4 5 7 8 0]
```
a1:make的时候指定一个size则len和cap都等于这个值

b1:初始化slice，len和cap都等于初始化元素的数量
b2:在b1的基础上append一个元素，此时slice长度为3超过b1的cap值2且没有超过b1的cap值2倍，因此新创建一个slice，新的slice cap为b1 cap的2倍4，len为3
b3:b3与b2同理，len为3 cap为4，b2和b3都是新分配一块地址出来的slice，所以b2和b3地址不相同
b4:在b1的基础上append元素超过b1 cap的2倍，因此b1的len和cap都等于实际元素个数
b5:同b4，但是在64位机器上当元素个数为奇数时，cap会在奇数上+1，所以b5的len为9，cap为10（没有32位的系统，没测试）
b7:与b2相对应，当b6的cap大于等于1024时，在b6的基础上append一个元素，需要的长度超过b6的cap且没有超过b6的cap值1.25倍，新slice的cap为b6 cap的1.25倍

c2、c3、c4有相同的地址，但是具体值不同，因为c3、c4都是在c2上append生成的，append的元素数量没有超过c2的cap，所以不会分配新的地址创建slice，c3 append的元素6被c4 append的7、8覆盖了，c2、c3、c4地址和数据是相同的，仅仅是len不同展示的数据不同

d1:在c3的基础上把len拉到与c3的cap相同，c3中也是有c3[6]为8这个值的且d1的地址与c3相同
d2:在c3的基础上截取第一个元素之后的数据，相对应c3，len和cap都删除了第一个数据，同时地址也改变了

## 2.defer return
``` code
func main() {
	log.Println("method a:", a())
	log.Println("method b:", b())
	log.Println("method c:", c())
}

func a() int {
	var i int
	defer func() {
		i++
		fmt.Println("a defer2:", i) // 打印结果为 defer: 2
	}()
	defer func() {
		i++
		fmt.Println("a defer1:", i) // 打印结果为 defer: 1
	}()
	return i
}

func b() (i int) {
	num := 0
	defer func() {
		i++
		fmt.Println("b defer2:", i) // 打印结果为 defer: 2
	}()
	defer func() {
		i++
		fmt.Println("b defer1:", i) // 打印结果为 defer: 1
	}()
	return num // 或者直接 return 效果相同
}

func c() (i int) {
	defer func() {
		i++
		fmt.Println("c defer2:", i) // 打印结果为 defer: 2
	}()
	defer func() {
		i++
		fmt.Println("c defer1:", i) // 打印结果为 defer: 1
	}()
	return i // 或者直接 return 效果相同
}
```
输出
``` output
a defer1: 1
a defer2: 2
2020/07/25 17:13:50 method a: 0
2020/07/25 17:13:50 method b: 2
2020/07/25 17:13:50 method c: 2
b defer1: 1
b defer2: 2
c defer1: 1
c defer2: 2
```
return并非原子操作，分为赋值和返回两个操作，defer return执行顺序为先执行return语句，写入返回值，再执行defer，defer执行完成再返回数据。
a方法写入返回值后，修改i的值已经不影响最终返回值；
b方法return写入返回值到i，defer方法修改i的值，最终返回；
c方法return写入b到返回值i，defer方法修改i的值，最终返回。

## 3.局部变量是否可以返回指针到上层
可以（第一次被问这个问题蒙了，以为是什么高端的问题，没反应过来，日常编程天天都在这样用）