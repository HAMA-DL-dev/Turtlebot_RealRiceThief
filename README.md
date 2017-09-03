# Turtlebot_RealRiceThief

# 레알밥도둑 터틀봇 - 간장게장
![teamlogo](/colorlogo.png)

조원 : [이도규](https://github.com/ldkl123), 정민재, [정현철](https://github.com/junghyeonchiul), [조민수](https://github.com/KoG-8)

## 0. 목차
1. 실행에 앞서

2. 실행법

3. 부분별 세부 설명


## 1. 실행에 앞서
**1.1. 사용한 하드웨어**

기본 터틀봇 구성품에 더해서 아래 품목을 추가로 사용했습니다.
 
현재 라즈베리파이 보드를 추가하여 2개를 사용하기 위한 작업 도중이며 아래 코드는 라즈베리파이 1개 기준입니다.

  + Raspberry Pi
  
  + Raspberry Pi Camera
  
  + Raspberry Pi Camera - Fisheye Lens (LS-40180)
  
  + Logitech C920 USB Webcam
  
  + 2x HC-SR04 Ultrasonic Sensor



**1.2. 사용한 소프트웨어**

  + ROS Kinetic

  + Python 2
  
  + OpenCV 3
  
  + Arduino Firmware
  
  + Ubuntu 16.04

**1.3. 사용한 오픈소스**
  
  + [raspicam_node](https://github.com/fpasteau/raspicam_node) by [fpasteau](https://github.com/fpasteau)


## 2. 실행법 
2.1. 이렇게

2.2. 저렇게

2.3. 하면

2.4. 됩니다


## 3. 부분별 세부 설명

### 3.1. 알고리즘
알고리즘설명

### 3.2. 라인트레이싱
~~~
codeblock
~~~

설명

### 3.3. 신호등
~~~
codeblock
~~~

설명

### 3.4. 표지판
~~~
codeblock
~~~

설명

### 3.5. 차단바
~~~
codeblock
~~~

설명

### 3.6. 터널
~~~
#!/usr/bin/env python
import rospy
import numpy
from sensor_msgs.msg import LaserScan
from wall.msg import raw_sensor_8_10
import copy
rospy.init_node('sensor', anonymous=True)
pub = rospy.Publisher('/raw_sensor', raw_sensor_8_10, queue_size = 5)
rate = rospy.Rate(10)

global data2
global copy_data
global count

raw = raw_sensor_8_10()
data2 = [0,0,0,0,0]
copy_data = [0,0,0,0,0]
count = 0

##############################################################################
####  Accept Lidar values and process the values (main)  #####################
##############################################################################

def callback(data):
	global data2
	global copy_data
	SensorData = data.ranges       # Get the data from Lidar
	

	Front = SensorData[1:18]+SensorData[342:359]
	real_Front = SensorData[1:5] + SensorData[354:359]
	Left = SensorData[54:90]
	Front_Left = SensorData[18:54]                   # Seperate the data depend of direction of turtlebot
	Front_Right = SensorData[306:342]
	Right = SensorData[270:306]
	
	Left_side = SensorData[85:95]
	Right_side = SensorData[265:275]
	
	Left_Beside = SensorData[60:70]
	Right_Beside = SensorData[290:300]

	back = SensorData[135:225]
	
	Front_Left_avg = avg(Front_Left)
	Front_Right_avg = avg(Front_Right)
	Left_avg = avg(Left)				# Each data has least 10 values including error values like coming up zero value
	Right_avg = avg(Right)				# so if error occured, except error values and make the average of their data			
	Left_Beside_avg = avg(Left_Beside)		
	Right_Beside_avg = avg(Right_Beside)
	Left_side_avg = avg(Left_side)
	Right_side_avg = avg(Right_side)
	Front_avg = avg(Front)
	Back_avg = avg(back)
	real_Front_avg = avg(real_Front) 

##############################################################################
####  Save the process values in messege form  ##################################################################################
#################################################################################################################################
	
 raw.data = [Front_avg, Left_avg, Front_Left_avg, Front_Right_avg, Right_avg, Left_Beside_avg, Right_Beside_avg, Back_avg, real_Front_avg, Left_side_avg, Right_side_avg]	

	raw.sharp = [SensorData[40], SensorData[29], SensorData[20], SensorData[9], SensorData[350], SensorData[339], SensorData[329], SensorData[320]]
	
	raw.front = [avg(SensorData[37:40]), avg(SensorData[33:35]), avg(SensorData[26:29]), avg(SensorData[23:25]), avg(SensorData[16:18]), avg(SensorData[13:15]), avg(SensorData[344:346]), avg(SensorData[338:340]), avg(SensorData[332:335]), avg(SensorData[330:334]), avg(SensorData[329:331]), avg(SensorData[321:324]) ]
	
#################################################################################################################################
	
 data2 = [Front_Left_avg, Front_avg , Front_Right_avg]

	copy_data = copy.copy(data2)

	data2.sort(reverse=True)

	find_same(data2[0])
	find_same(data2[1])
	
	rospy.loginfo(raw.front)
	pub.publish(raw)

####  main ends #################################################################################################################





#################################################################################################################################
####  Functions  ################################################################################################################
#################################################################################################################################

def avg(data):
	return 	sum(data)/(len(data)-data.count(0)+0.01)           # It makes several value's average




def find_same(data1):
	global data2
	global copy_data
	global count

	
	for i in range(0, len(copy_data)):
		if copy_data[i] == data1:
			if count ==0:
				raw.Flag = i+1
				raw.Far = data1
				pub.publish(raw)
				count = 1
			else: 
				raw.Flag2 = i+1
				pub.publish(raw)
				count = 0


def scan_sensor():
	rospy.Subscriber('/scan', LaserScan, callback)
	rospy.spin()			  
	
if __name__ == '__main__':
	try:	
		scan_sensor()
	except rospy.ROSInterruptException:
		pass



#!/usr/bin/env python
import rospy
import numpy
from geometry_msgs.msg import Twist
from nav_msgs.msg import Odometry
from wall.msg import raw_sensor_8_10
import sys, select, termios, tty
import copy
from math import atan
from math import asin
import math

global Flag
global check

global flag
global flag2
global count

global num

global dot
global past
global pose
global copy_pose
global past_number

global bool_way



bool_way = [0,0]

num=0
count = 0
dot = [0, 0]
past = [[0,0]]
pose = [0, 0]
copy_pose = [0,0]
past_number = [0]
flag =0
flag2 =0
Flag = 0
check = 0




#################################################################################################################################
####  Publish the process values to '/cmd_vel' node  ############################################################################
#################################################################################################################################

pub = rospy.Publisher('/cmd_vel', Twist, queue_size = 5)
rospy.init_node('vel', anonymous=True)
rate = rospy.Rate(10)
twist = Twist()

#################################################################################################################################



#################################################################################################################################
####  Exit node  ################################################################################################################
#################################################################################################################################

def myhook():               # If Ctrl + c occured, the node ends by myhook function and turtlebot's velocity returns to 0.
	twist = Twist()
	twist.linear.x=0
	twist.angular.z=0
	pub.publish(twist)

#################################################################################################################################


#################################################################################################################################
####  main   ####################################################################################################################
#################################################################################################################################

def callback1(data):
	global Flag
	global flag2
	global past
	global count
	global pose
	global copy_pose
	global past_number
	global check
	global bool_way	

	rospy.on_shutdown(myhook)
	Lidar = data.data
	Lidar2 = data.sharp
	Lidar3 = data.front			# Get the processed values from sensor node
	Far = data.Far
	Flag = data.Flag
	
	wall = 0.16				# minimum value of distance from wall turtlebot have to aviod 
	scan_wall2(Lidar2)
	remind_past()				# Consider present position is visited place or not
	
	
	if count == 1:
		while check < 10000:		# If It placed on visited place, turn way back 
			print('visited place')
			destination1(Flag)
			check = check+1
		check=0
		count = 0


	
	
	if scan_wall(Lidar3) == 1:		# If there is wall in front of turtlebot or block way, turn back
		rospy.loginfo('strange wall')
		while check <= 100:
			destination5(3)	
			check = check+1
		check=0	
		rospy.loginfo('End')
	else:	
		if bool_way[0] == 1:		# If there is wall beside the turtlebot
			if Lidar[8] <= wall+0.03 and Lidar[8] > 0: 	# If turtlebot is in uncertain position with blocked wall
				store_pose()
				remind_past()
			
			
				if Lidar[9] > Lidar[10]:		# Compare the left and right side of distance and turn to long distance
					while check<= 100:
						destination5(1)
						rospy.loginfo('turn_Left')
						check = check+1
					check = 0
				else:
					while check <= 100:
						destination5(3)
						rospy.loginfo('turn_Right')
						check = check+1
					check = 0
				
			else:
				while check <= 50:
					destination4(bool_way[1])
					check = check+1
					
				check = 0
				bool_way[0] = 0		# Refresh the values
				bool_way[1] = 0
				
				
				
		else:	
			# If there is no blocks around turtlebot, It goes to exit way using callback4 function
			flag2=0				# Refresh the flag number to store the coordinate again
			callback4()			
			rospy.loginfo('check')

	
	rospy.loginfo('######################################################################')


####  main ends   ###############################################################################################################



#################################################################################################################################
####  Functions  ################################################################################################################
#################################################################################################################################

def scan_wall(data):
	counting = 0
	for i in range(0, len(data)):
		if data[i] < 0.2 and data[i] > 0:       # Find the distances which are less 0.2m and counts the number of data 
			counting = counting+1

	
	
	if counting >= 6:				# If the count number of values over 6, turtlebot consider that there is wall
		return 1				# in front of It and returns 1			
	else:
		return 0
	

def scan_wall2(data):
	global bool_way                           
	temp = 0
	number = 0

	for i in range(0, len(data)):			# Accept the front Lidar values and excepting zero value and select the init value 
		if data[i] != 0:
			temp = data[i]
			break	
	
	for j in range(i, len(data)):
		if data[j] < temp and data[j] !=0:	# Compare the values and get the smallest value 
			temp = data[j]
			number = j	
	#rospy.loginfo([temp, number])
	if temp <= 0.20 and number <= 3:
		bool_way[0] = 1				# the smallest value means the wall's postion 
		bool_way[1] = 3				# Return the direction the turtlebot have to go to avoid the wall
		#rospy.loginfo('Right')		
	elif temp <= 0.23 and number > 3:
		bool_way[0] = 1
		bool_way[1] = 1
		#rospy.loginfo('Left')
	

def store_pose():
	global past
	global flag2
	global pose
	global count	
	global copy_pose
	global past_number
	global num 	
	
	if count == 0:
		if flag2==0:
			copy_pose = copy.copy(pose)	# Store the place coordinate which turtlebot must avoid 
			past.append(copy_pose)		# ex) entrance of tunnel
			past_number.append(0)
			num = num+1
			flag2 = 1

		

def remind_past():
	global past
	global pose
	global count
	global past_number


	for i in range(0, len(past)):
			X = abs(pose[0]-past[i][0])	# Subtract the Stored coordinate and the present coordinate 
			Y = abs(pose[1]-past[i][1])
			
			if  X > 0.1 and Y > 0.1:
				#rospy.loginfo('saved')		# After the coordinate stored, turtlebot have to get out the stored position 
				past_number[i] = 1		# so coordinate subtractin over 0.1m makes the past_number 1 	
	
			if  X < 0.1 and Y < 0.1:
				#rospy.loginfo('First step')	# When turtlebot come back to past coordinate make count 1 
				if past_number[i] == 1:		# Count 1 make the turtlebot turns for optional time
					rospy.loginfo([X, Y])
					past_number[i] = 0
					count = 1
			
				
#################################################################################################################################
####  Turtlebot's linear, angular velocity funcation ############################################################################
#################################################################################################################################


def destination1(data1):
	if data1==1:	
		twist.linear.x = 0.0
		twist.angular.z = 0.6
		pub.publish(twist)
	elif data1==3:
		twist.linear.x = 0.0
		twist.angular.z = -0.6            #Right (-) Left (+) 
		pub.publish(twist)


def destination4(data1):
	if data1==1:	
		twist.linear.x = 0.04
		twist.angular.z = 0.5
		pub.publish(twist)
	elif data1==3:
		twist.linear.x = 0.04
		twist.angular.z = -0.5            #Right (-) Left (+) 
		pub.publish(twist)
	
def destination5(data1):
	if data1==1:	
		twist.linear.x = -0.01
		twist.angular.z = 0.6
		pub.publish(twist)
	elif data1==3:
		twist.linear.x = -0.01
		twist.angular.z = -0.6            #Right (-) Left (+) 
		pub.publish(twist)
				
#################################################################################################################################


#################################################################################################################################
#### Converting Global coordinate to Local coordinate  ##########################################################################
#################################################################################################################################

def callback3(data):
	global dot
	global flag
	global pose
	global past
	global copy_pose
	global init_ori
	global init_deg
	global angle_goal
	global angle_twisted
	global ori
	global local_pose
	global global_pose
	
	local_pose = [0,0]
	global_pose = [0,0]
	
	if flag ==0:
		dot[0] = data.pose.pose.position.x    		# Using flag to save the initial coordinate which turtlebot starts just one time
		dot[1] = data.pose.pose.position.y
		past[0] = dot	
		copy_pose = copy.copy(dot)			# Copy_pose save the coordinate in past_postition to avoid to heading to 									  entrance
		flag = 1
	
	
	pose[0] = data.pose.pose.position.x			# Updating the coordinate 
	pose[1] = data.pose.pose.position.y 
	
	if flag==1:
		init_deg = math.degrees(math.pi/2-2*asin(data.pose.pose.orientation.z))  # Use flag to save the initial degree from global axis
		if init_deg < 0:
			init_deg = 360 + init_deg		# Convert the degree to 0 ~ 360
		init_ori = math.radians(init_deg)
		flag=2
	
	global_pose[0] = pose[0] - dot[0]			# Parallel translate make initial coordinate to [0,0]
	global_pose[1] = pose[1] - dot[1]	
	 
	
	
	
	ori = math.degrees(math.pi/2-2*asin(data.pose.pose.orientation.z))
	
	if ori < 0:
		ori = 360 + ori					# Present degree while turtlebot is moving and Convert the degree to 0 ~ 360 
	
	ori = ori - init_deg
	
	if ori < 0:
		ori = 360 + ori

	# Using initial degree from global axis to make the rotation matrix 

	matrix_angle = [[math.cos(init_ori), -math.sin(init_ori)],[math.sin(init_ori), math.cos(init_ori)]]	
		
	# Multiply the rotation matrix to present global coordinate and make local coordinate

	local_pose[0] = matrix_angle[0][0]*global_pose[0] + matrix_angle[0][1]*global_pose[1]
	local_pose[1] = matrix_angle[1][0]*global_pose[0] + matrix_angle[1][1]*global_pose[1]
	
	# Calculate the degree turtlebot have to turn and head depend on exit's direction

	angle_goal = math.degrees(atan((2-local_pose[1])/(2+local_pose[0])))

#################################################################################################################################




#################################################################################################################################
#### Making turtlebot turn for heading to exit coordinate  ######################################################################
#################################################################################################################################


def callback4():
	rospy.on_shutdown(myhook)
	
	speed = 0.09
		
	
	if ori < 270 + angle_goal and ori > 90 + angle_goal:	# Catch the turtlebot's present heading and judge which why to turn
		if abs(ori - 270 - angle_goal) < 6:		# If Present degree and desire degree's substraction is lower than 6
			twist.linear.x = speed
			twist.angular.z = -0.0
			pub.publish(twist)
		else:
			twist.linear.x = speed
			twist.angular.z = -0.3
			pub.publish(twist)
	else:
		if abs(ori - 270 - angle_goal) < 6:	
			twist.linear.x = speed
			twist.angular.z = -0.0
			pub.publish(twist)
		else:
			twist.linear.x = speed
			twist.angular.z = 0.3
			pub.publish(twist)


def listen():
	rospy.Subscriber('/odom', Odometry, callback3)
	rospy.Subscriber('/raw_sensor', raw_sensor_8_10, callback1)
	rospy.spin()


if __name__ == '__main__':
	listen()
~~~



설명



