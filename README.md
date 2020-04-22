背景是这样的，有一次我的服务器突然挂掉了，是由于cpu占用过高，但是我又不知道是哪个进程造成的。于是便想找一个监控工具来监控我的机器，我希望这个工具足够简单，没有复杂的配置，开箱即用，能够监控到当cpu负载过高时，发生的时间以及当时是什么进程造成这样的问题。很可惜，我并没有找到这样的工具（若有这样方便的工具请告知我），但我实在不想因此而给服务器上一套很重的监控系统。于是乎，我决定自己写一个工具。思路如下：

每隔一段时间就查看cpu占用率，当超过了设置的百分比，就触发记录；
调用top命令，按照cpu占用率从高到低排列，将输出记录到文件中，文件以时间命令；
程序能后台运行。
针对第一点，我用了一个库：github.com/shirou/gopsutil，里面有关于cpu使用率以及内存使用率的相关方法，比较简单。

代码如下：

func visor() {
    for {
        per, _ := cpu.Percent(time.Duration(config.GetConfig().Interval)*time.Second, false)
        if per[0] > config.GetConfig().AlterLimit {
            record(per[0])
        }
    }
}

func record(per float64) {
    now := time.Now().Format("2006-01-02-15-04-05")
    logFile := config.GetConfig().SnapPath + now + ".log"
    f, _ := os.Create(logFile)
    defer f.Close()

    f.Write([]byte{'\n'})
    f.Write([]byte("此时cpu占用率：" + strconv.FormatFloat(per, 'f', 10, 64)))
    f.Write([]byte{'\n'})
    topCmd := exec.Command("top", "-H", "-n", "1", "-b")
    topOutput, _ := topCmd.Output()
    f.Write(topOutput)
}

使用方法示例如下：

# 后台启动
visor -start -config=./config/config.yaml -d
# 结束进程
visor -stop
