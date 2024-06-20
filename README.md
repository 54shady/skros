# Learn ROS

[how to guide ros 101](https://clearpathrobotics.com/blog/2014/01/how-to-guide-ros-101/)

[ros guides](http://www.clearpathrobotics.com/assets/guides/melodic/ros/)

## baisc

ros的包管理rospack(查询是否安装某个包)

	rospack list | grep rospy

	https://index.ros.org/p/rospy/#noetic-overview

## 启动实验环境

运行安装好ros环境的容器

	drun --name rosdev -v $PWD:/ws ros:noetic-ros-base /bin/bash

设置一下python的路径

	ln -s /usr/bin/python3 /usr/bin/python

## HelloRobot

在终端1中运行roscore

	source /opt/ros/$ROS_DISTRO/setup.bash
	roscore

在终端2中运行(docker exec -it rosdev /bin/bash)

	source /opt/ros/$ROS_DISTRO/setup.bash
	rostopic list

运行一个名为hello的publisher,发出字符串"Hello Robot"

	rostopic pub /hello std_msgs/String "Hello Robot"

在终端3中运行(docker exec -it rosdev /bin/bash)

	source /opt/ros/$ROS_DISTRO/setup.bash

查看到hello的node

	rostopic list
	/hello
	/rosout
	/rosout_agg

	rostopic info hello

		Type: std_msgs/String

		Publishers:
		 * /rostopic_3879_1718873948805 (http://4873126ff62d:42393/)

		Subscribers: None

打印出publisher的字符串

	rostopic echo /hello

## Writing Publisher and Subscriber

[Writing Publisher and Subscriber](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29)

安装好catkin后配置对应的环境变量

	source /opt/ros/$ROS_DISTRO/setup.bash

在容器中创建工作空间

	mkdir -p ~/myws/src
	cd ~/myws/
	catkin_make
	source ~/myws/devel/setup.bash

配置完环境后会设置如下环境变量

	echo $ROS_PACKAGE_PATH
	/root/myws/src:/opt/ros/noetic/share

创建名为beginner_tutorials的包(std_msgs rospy roscpp是包的依赖)

	cd ~/myws/src
	catkin_create_pkg beginner_tutorials std_msgs rospy roscpp

编译包后配置对应的环境变量

	cd ~/myws
	catkin_make
	source ~/myws/devel/setup.bash

进入到包的目录

	roscd beginner_tutorials
	mkdir scripts && cd scripts

添加talker和listener到当前目录

	cp /ws/talker.py .
	cp /ws/listener.py .
	chmod +x talker.py
	chmod +x listener.py

再次编译node

	cd ~/myws
	catkin_make

运行node(首先要运行roscore)

在终端1中运行roscore

	roscore

在终端2中运行名为talker的publisher(docker exec -it rosdev /bin/bash)

	cd ~/myws
	source ./devel/setup.bash

	rosnode list //这里可以先查看下node的情况
	/rosout

	rosrun beginner_tutorials talker.py

在终端3中运行名为listener的subscriber(docker exec -it rosdev /bin/bash)

	cd ~/myws
	source ./devel/setup.bash

	rosnode list //这里可以先查看下node的情况
	/rosout
	/talker_1199_1718864749032

	rosrun beginner_tutorials listener.py

或者使用ros里自带的通用的listener(rostopic)

	cd ~/myws
	source ./devel/setup.bash
	rostopic echo chatter

## Using msg

	roscd beginner_tutorials
	mkdir msg
	echo "string firstName" > msg/myNum.msg
	echo "string secondName" >> msg/myNum.msg
	echo "int8 age" >> msg/myNum.msg
	echo "int32 score" >> msg/myNum.msg

将文件package.xml中下面两行注释去掉

	<build_depend>message_generation</build_depend>
	<exec_depend>message_runtime</exec_depend>

使用包管理器查看first-order的依赖

	rospack depends1 beginner_tutorials

在CMakeLists.txt添加message_generation

	find_package(catkin REQUIRED COMPONENTS
	  roscpp
	  rospy
	  std_msgs
	  message_generation
	)

	...

	add_message_files(
		FILES
		myNum.msg
	)

	generate_messages(
		DEPENDENCIES
		std_msgs
	)

查看是否修改成功

	rosmsg show beginner_tutorials/myNum
	rosmsg show myNum

## Using srv

	roscd beginner_tutorials
	mkdir srv

添加一个服务文件(两个int request, 一个int reponse)

	echo "int64 a" > srv/AddTwoInts.srv
	echo "int64 b" >> srv/AddTwoInts.srv
	echo "---" >> srv/AddTwoInts.srv
	echo "int64 sum" >> srv/AddTwoInts.srv

	rossrv show beginner_tutorials/AddTwoInts
	rossrv show AddTwoInts

修改CMakeLists.txt如下后编译代码

	add_service_files(
	  FILES
	  AddTwoInts.srv
	)

编译代码

	cd ~/myws/
	catkin_make

## Writing Service Client(依赖上面的AddTwoInts.srv)

[WritingServiceClient](http://wiki.ros.org/rospy_tutorials/Tutorials/WritingServiceClient)

	roscd beginner_tutorials

修改CMakeLists.txt如下

	catkin_install_python(PROGRAMS
	  scripts/add_two_ints_server.py
	  scripts/add_two_ints_client.py
	  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
	)

在终端1中运行roscore

	roscore

在终端2中运行server(docker exec -it rosdev /bin/bash)

	cd ~/myws
	source ./devel/setup.bash
	rosrun beginner_tutorials add_two_ints_server.py

在终端3中运行client(docker exec -it rosdev /bin/bash)

	cd ~/myws
	source ./devel/setup.bash
	rosrun beginner_tutorials add_two_ints_client.py 11 22
