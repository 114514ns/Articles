使用go语言操作docker。并在容器内部执行命令，会使用Client.ContainerExecAttach方法，然而发现大部分命令无法执行，网上也没找到解决方法。

![](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/20241005220613.png)

花了一点时间才发现需要指定Tty=true

~~~go
	execConfig := types.ExecConfig{
		Cmd:          cmd,
		AttachStdout: true,
		AttachStderr: true,
		User:         "root", // 确保使用 root 执行命令
		Tty:          true,
	}
~~~

完整代码

~~~go
func runCommand(command string, path string, containerId string) string {
	cmd := []string{}

	var arr = strings.Split(command, " ")

	for i := range arr {
		cmd = append(cmd, arr[i])
	}
	//cmd = append(cmd, path)

	execConfig := types.ExecConfig{
		Cmd:          cmd,
		AttachStdout: true,
		AttachStderr: true,
		User:         "root", // 确保使用 root 执行命令
		Tty:          true,
	}

	// 创建执行命令
	execIDResp, _ := dockerClient.ContainerExecCreate(context.Background(), containerId, execConfig)

	// 执行命令
	resp, err := dockerClient.ContainerExecAttach(context.Background(), execIDResp.ID, types.ExecStartCheck{})
	if err != nil {
		log.Fatalf("Failed to attach to exec instance: %v", err)
	}
	defer resp.Close()

	// 读取输出
	var output bytes.Buffer
	_, err = stdcopy.StdCopy(&output, nil, resp.Reader)
	return output.String()
}
~~~

