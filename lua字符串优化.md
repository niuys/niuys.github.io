  字符串是一个永恒的问题。字符串一般用作两种用途，一种是普通字符串，有结尾符，一种是本质是二进制缓冲区，必须带长度信息。lua里有userdata可以专用来做二进制缓冲区，当然用字符串也没问题，但是userdata应该是一个更优的选择，更极端一点用lightuserdata，这个要自己负责内存的申请和释放，还要注意lua的error跳转，风险较大。
  
  在一个系统里，代码自身的字符串的数量是可控的，不可控的是两种：一种是日志输出，一种是通讯协议，还有一种是代码里做的字符串拼接，尤其是在循环里。前两者应该尽量避免在lua层做字符串操作，而是把数据放到c层，lua层只提供数据和获得必要的返回值，不使用lua的字符串。skynet里的日志接口skynet.error使用里tostring和table.concat来获取日志信息，把这部分工作放到c里做，直接发送到日志服务。当然这样的问题是绕过里metatable的tostring，鉴于日志的量和大部分情景，这么做是值得的。在定时器接口用到了把时间间隔tostring成字符串，这个量很大，实际上时间间隔的值是离散有限的，做了一个string cache，这样减少了tostring的操作。
  
lua的字符串操作tostring是相对昂贵的，因为涉及到metatable，用sprintf转字符串也是代价较高。string是参与到内存回收的。所以从性能来讲，尽量避免运行时引入一次性的和逻辑无关的字符串，是一个可以考虑的优化点。
