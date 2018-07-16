# 303_shopping_car_development_document_2
303's big project:shopping car's development document

minipc搭建rikirobot小车环境

一、解决Ubuntu 16.04 SSH 无法远程登录问题(没问题可以先不管)
	
	1、安装 open ssh：
		sudo apt-get install openssh-server
	2、修改root密码
		sudo passwd root
	3、辑配置文件，允许以 root 用户通过 ssh 登录：
		sudo vi /etc/ssh/sshd_config
		找到：PermitRootLogin prohibit-password禁用
		添加：PermitRootLogin yes
		sudo service ssh restart
		OK,正常登录！！！

二、minipc及树莓派的ubuntu16.04安装ros kinetic
	1、百度 ros kinetic

	找到这个：kinetic/Installation/Ubuntu - ROS Wiki 就这个
	找到 kinetic/Installation - ROS Wiki就点这个，然后再点ubuntu,因为是ubuntu的。

	2、然后按照安装步骤一步一步下去

		2.1	sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
				建立源的列表
		2.2	sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
				建立密匙
		2.3	sudo apt-get update
				更新包
		2.4	sudo apt-get install ros-kinetic-desktop-full
				安装ros-kinetic全桌面版，有ROS，rqt，rviz,robot-generic libraries,2D/3D simulators,navigation 和 2D/3D perception
		2.5	sudo rosdep init
				初始化rosdep

		2.6	rosdep update
				更新rosdep
		2.7	echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
				建立环境
		2.8	source ~/.bashrc
				更新环境
		2.9	使用roscd验证一下，有没有所有列表出来

	3、虚拟机（Ubuntu 16.04 + ROS Kinetic ）端开发环境构建

	准备资料：
		拉过来的目录都要进行权限解放：在home目录下：chmod -R 777 catkin_ws/  以及 chmod -R 777 Work/
	3.1 Rikirobot_setup源码 树莓派Work目录下拷贝。
		3.1.1 ./dev_tools_install.sh	开发工具依赖包
		3.1.2 ./ros_packages_install.sh	源码依赖包
		3.1.3 ./rikihost_network.sh	ROS网络环境设置

	3.2 rikirobot_project源码，树莓派catkin_ws/src目录下拷贝
		3.2.1 在虚拟机端 mkdir ~/catkin_ws/src -p
		3.2.2 进入源码目录 cd ~/catkin_ws/src 把rikirobot_project源码拷贝到这里
		3.2.3 返回上层目录 cd ~/catkin_ws  下开单线程编译    catkin_make -j1

		编译错误解决办法：1、缺失depth_camera包（深度摄像头），如果没用到，可以直接删掉
			             2、缺失teleop包（遥控的），应该是缺少libusb，缺少库，驱动没装，把ros_pack里面的echo "add ps3"及以下的注释去掉		

		3.2.4 vim ~/.bashrc 翻到最后查看是否有这样一行：source ~/catkin_ws/devel/setup.bash
				没有的话手动加进去也行
		3.2.5 在/catkin_ws/devel/ 目录下soucre一下：source ~/.bashrc
		3.2.6 编译完 执行  rospack profile 
		3.2.5 roscd rikir 再按tab键验证是否能补全，补全成功证明可以 
	
	3.3 然后ojbk

三、minipc还没配好bashrc文件，改了一点，虚拟机的bashrc





四、第一章：rikirobot小车的网络配置与控制

	1、修改.bashrc文件中的 export ROS_MASTER_URI=http://192.168.43.43:11311 ，ip地址，使虚拟机的节点指向树莓派
	改完之后，soucre ~/.bashrc 文件

	2、登录过去树莓派：ssh rikirobot@192.168.43.43	输入密码：123456
	登进之后，观察bash文件

	3、在树莓派启动小车：roslaunch rikirobot bringup.launch 
	   一直等到出现 Gyro calication complete...才可以控制小车
	   树莓派端控制小车：	a、再开一个中端，ssh过去
				b、rosrun teleop_twist_keyboard teleop_twist_keyboard.py 

       在虚拟机那边控制小车：	a、因为虚拟机ros的节点是指向树莓派的，所以可以控制。
				b、rosrun teleop_twist_keyboard teleop_twist_keyboard.py 

	4、打开rviz,一定要在虚拟机上打开，树莓派跑不动：rosrun rviz rviz
		点file--open config /home/rikirobot/catkin_ws/src/rikirobot_project/rikirobot/rviz/odometry.rviz



五、第二章：IMU的自动校正

	1、在树莓派那端启动小车：
		ssh rikirobot@192.168.43.43	输入密码：123456 
		roslaunch rikirobot bringup.launch 	一直等到出现 Gyro calication complete...才可以控制小车

	2、校正文件在树莓派那端，所以还要开一个中端，ssh过去树莓派 ：ssh rikirobot@192.168.43.43	输入密码：123456 
		
		roscd rikirobot 
		cd param/
		cd imu/

		这个是查看imu的数据

		rostopic echo /imu/data

		这个是校正imu的数据

		rosrun imu_calib do_calib
		
		校正开始，第一次为X+方向，命令行提示按回车，第二次为X-方向，命令行提示按回车
			  第三次为Y+方向，命令行提示按回车，第二次为Y-方向，命令行提示按回车
			  第一次为Z+方向，命令行提示按回车，第二次为Z-方向，命令行提示按回车

		Done.

		输入命令：ll 查看 imu_calib.yaml 更新的时间是不是当前时间

		返回上一个中端，重启 bringup.bash 文件: roslaunch rikirobot bringup.launch

		ojbk了，可以查看imu的数据：rostopic echo /imu/data 发现误差减少了


六、第三章：校正rikirobot的线速度和角速度

	1、在树莓派那端启动小车：
		ssh rikirobot@192.168.43.43	输入密码：123456 
		roslaunch rikirobot bringup.launch 	一直等到出现 Gyro calication complete...才可以控制小车


	2、再开一个中端，ssh过去树莓派 ：ssh rikirobot@192.168.43.43	输入密码：123456 
		rosrun rikirobot_nav calibrate_linear.py
	
	3、开中端，是虚拟机这端， 打开 rqt : rosrun rqt_reconfigure rqt_reconfigure 

	打开之后，你就会看到一个校正界面，第二步因为rosrun了calibrate_linear.py（线速度），在左边就有一个calibrate_linear,点开，就出现一个线速度的校正界面

		test_distance			--测试距离
		speed				--速度
		tolerance			--容忍误差
		odom_linear_scale_correction	--实际计算到达的距离
		strat_test			--开始测试

	。。。。。。。。。。有空再写。。。。。。	


	
七、第四章：SLAM构建地图

	1、在树莓派那端启动小车：
		ssh rikirobot@192.168.43.43	输入密码：123456 
		roslaunch rikirobot bringup.launch 	一直等到出现 Gyro calication complete...才可以控制小车


	2、再开一个中端，ssh过去树莓派 ：ssh rikirobot@192.168.43.43	输入密码：123456 
		打开雷达：roslaunch rikirobot lidar_slam.launch

	3、再开一个中端，在虚拟机这端，打开rviz：rosrun rviz rviz
		点击file,openconfig,riki\...\slam.rviz


八、第五章：地图自动导航

	
	1、在树莓派那端启动小车：
		ssh rikirobot@192.168.43.43	输入密码：123456 
		roslaunch rikirobot bringup.launch 	一直等到出现 Gyro calication complete...才可以控制小车

	2、再开一个中端，ssh过去树莓派 ：ssh rikirobot@192.168.43.43	输入密码：123456 
		打开：roslaunch rikirobot navigate.launch

	3、再开一个中端，在虚拟机这端，打开rviz：rosrun rviz rviz
		点击file,openconfig,riki\...\navigate.rviz

	4、使用2D pose 调整位姿














