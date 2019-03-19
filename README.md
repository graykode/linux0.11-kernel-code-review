linux0.11-kernel-code-review
==========

- The old Linux kernel source ver 0.11 review with line by line for OS lecture. Original code is [here](https://github.com/yuanxinyu/Linux-0.11). More detail information about kernel code review, I recommend [this book](https://www.amazon.com/Art-Linux-Kernel-Design/dp/1466518030). I quoted the table of contents of the book to explain.
- This is just repository for review when i listen OS(Operation System), in this semester to remember easily.



## Build on Linux

* a linux distribution: **debian, ubuntu and mint with GUI** are recommended, Ubuntu 16.04  GUI version with VM VirtualBox

1. `sudo apt-get update && sudo apt-get upgrade`
2. `sudo apt-get install build-essential`
3. `sudo apt-get install qemu`
4. `make`
5. `make start`



## Overall(I drawed)

### 1. Overall Memory Layout in Linux Kernel 0.11

![](https://lh6.googleusercontent.com/vY7zneoCXAlbQJDrhb-qiZIqDzC2HGtXZgTnHl2kdRr2CXrQVJ-pa6rcuvYUNkyb0qEwY8GqZHDOjg=w1920-h867-rw)



### 2. More Detail Overall Memory Layout in Linux Kernel 0.11





![](https://lh5.googleusercontent.com/EVnEtteNeW1Nxx6WBcVU8EOYzKVq5-l7YNZACyzeq-e6GSlUJloo4nszG6xqvHmVeCGlyFZQJU9Hew=w1920-h867-rw)



### 3. File System

![](https://lh6.googleusercontent.com/ReDu1y0JFoKvzgBdJj_PHYiW2-aEhF6M1v9RvyzIvO0O34pLNSyFNCfZjTpKgOAKbENc46l0tBLJ5A=w1920-h867-rw)



### 4. Segmentation and Paging

![](http://sarghis.com/blog/wp-content/uploads/2012/11/a0051744_4df1bef792239.jpg)

- image refer to [here](http://sarghis.com/blog/637/)



## License

<p xmlns:dct="http://purl.org/dc/terms/">
  <a rel="license"
     href="https://creativecommons.org/licenses/by-nc-sa/2.0/kr/">
    <img src="https://wikidocs.net/static/img/by-nc-sa.png" style="border-style: none;" alt="CC0" />
  </a>
</p>



## Code Review Author

- Name : Tae Hwan Jung(@graykode)
- Email : nlkey2022@gmail.com
