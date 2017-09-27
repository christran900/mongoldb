# I. Mở đầu
## 1.1. Giới thiệu
<ul>
Replica set là một hệ replication trong mongoDB. Tập data sẽ được nhân bản trên nhiều server khác nhau thay vì tập trung trên chỉ duy nhất 1 con server. Nhờ vậy, replica đáp ứng cao tính High avaibility và dự phòng. Nó cũng scale read request cho mogodb. Mô hình của replica set trong mongodb gần giống replication trong mysql
</ul>
## 1.2. Mô hình 
<img src="https://i.imgur.com/drfPWzE.png">
<ul>
Mô hình triển khai gồm 3 máy:
* máy đóng vai trò là primary: ubuntu server 16.04, địa chỉ IP: 192.168.1.36/24
* máy đóng vai trò là secondary: ubuntu server 16.04 địa chỉ IP: 192.16.1.48/24
* máy đóng vai trò là Arbitrator: ubuntu server 16.04 địa chỉ IP: 192.168.1.49/24
</ul>

# II. Triển khai 
## 2.1. Cài đặt mongodb
<ul>
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
	
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update
sudo apt-get install -y mongodb-org
</ul>
## 2.2. cài đặt mongodb.service
<ul>
	Tạo file mongodb.service trong thư mục /etc/systemd/system/
	với nội dung file như sau 

[Unit]
Description=High-performance, schema-free document-oriented database
After=network.target
 
[Service]
User=mongodb
ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf
 
[Install]
WantedBy=multi-user.target

Tắt dịch vụ và bắt đầu tiến hành cấu hình
sudo service mongodb stop
</ul>
## 2.3. Cấu hình mongodb replica
<ul>
	sửa hostname: sudo hostname mongo1.appconus.com
	truy cập vào file và sửa file hostname: sudo vim /etc/hostname --> mongo1.appcounus.com
	truy cập vào file /etc/hosts và sửa thành như sau 


sudo vim /etc/hosts
127.0.0.1 localhost mongo1.appconus.com
192.168.1.36 mongo1.appconus.com
192.168.1.48 mongo2.appconus.com
192.168.1.49 mongo3.appconus.com


Cấu hình mongodbreplica
Truy nhập vào thư mục /etc/mongodb.conf

#mongod.conf                                                               
 
#for documentation of all options, see:                                    
#http://docs.mongodb.org/manual/reference/configuration-options/         
 
#Where and how to store data.                                              
storage:                                                                    
  dbPath: /var/lib/mongodb                                                  
  journal:                                                                  
    enabled: true                                                          
 
#where to write logging data.                                              
systemLog:                                                                  
  destination: file                                                         
  logAppend: true                                                           
  path: /var/log/mongodb/mongod.log                                         
 
#network interfaces                                                        
net:                   
  bindIp: [127.0.0.1, 192.168.1.36] #thêm địa chỉ IP tương ứng với server
  port: 27017 
 
replication:                                                           
   oplogSizeMB: 30                                               
   replSetName: appconus_rs #tên của replica

làm tương tự vs 2 server còn lại
</ul>
## 2.4. Sử dụng 
<ul>
	Tại primary khởi động mongodb
	sudo service mongodb start 
	Mở mongo cli client: mongo
	thêm lệnh rs.initiate() để server này thành primary
	sau đó tiếp tục thêm secondary 
	rs.add('mongo2.appconus.com:2017')
	thêm arbitrator
	rs.add('mongo3.appconus.com:2017' true)

Kiểm tra thiết lập hệ thống 
	rs.config()
	kiểm tra trạng thái replica set 
	rs.status

Test hệ thống
ở primary nhập dữ liệu vào và kiểm tra  
db.chris.save({_id:2, value:'Appconus'})
	
db.chris.find()
	<img src="https://i.imgur.com/5TvACxk.png">

sang secondary kiểm tra 
	<img src="https://i.imgur.com/XZQHAYe.png">

 kiểm tra việc bầu cử
	tắt mongodb service trên primary và sang secondary kiểm tra 
	<img src="https://i.imgur.com/4baXAGc.png">
	quay trở về primary và bật dịch vụ mongodb lên xem trạng thái của primary lúc này 
	<img src="https://i.imgur.com/g1zfluR.png">
</ul>>
## 2.5. Cây thư mục cài đặt mongodb bằng ansible
<img src="https://i.imgur.com/Txn17Zz.png">
