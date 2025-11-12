# 🚀 AWS EC2-তে Dockerized Full-Stack প্রজেক্ট ডিপ্লয় করার সম্পূর্ণ গাইড

> **লেখক**: Sourab Hossain  
> **প্রজেক্ট**: Ostad-Docker  
> **শেষ আপডেট**: ২০২৫ নভেম্বর ১২  
> **ভাষা**: বাংলা  

---

## 📋 সূচিপত্র

1. [প্রজেক্ট অভিজ্ঞতা](#প্রজেক্ট-অভিজ্ঞতা)
2. [আর্কিটেকচার](#আর্কিটেকচার)
3. [প্রয়োজনীয়তা](#প্রয়োজনীয়তা)
4. [AWS EC2 সেটআপ](#aws-ec2-সেটআপ)
5. [Docker & Docker Compose ইনস্টল](#docker--docker-compose-ইনস্টল)
6. [প্রজেক্ট ক্লোন এবং সেটআপ](#প্রজেক্ট-ক্লোন-এবং-সেটআপ)
7. [Security Group কনফিগারেশন](#security-group-কনফিগারেশন)
8. [Docker Compose চালানো](#docker-compose-চালানো)
9. [API টেস্টিং](#api-টেস্টিং)
10. [ডাটাবেস ম্যানেজমেন্ট](#ডাটাবেস-ম্যানেজমেন্ট)
11. [সমস্যা সমাধান](#সমস্যা-সমাধান)
12. [চেকলিস্ট](#চেকলিস্ট)
13. [রক্ষণাবেক্ষণ এবং আপডেট](#রক্ষণাবেক্ষণ-এবং-আপডেট)

---

## 🎯 প্রজেক্ট অভিজ্ঞতা

এই প্রজেক্টে আছে:

| Component | প্রযুক্তি | পোর্ট | বর্ণনা |
|-----------|---------|------|--------|
| **Backend** | Node.js + Express | 5050 | REST API সার্ভার |
| **Frontend** | React + Vite | 5173 | ওয়েব ইউজার ইন্টারফেস |
| **Database** | MongoDB | 27017 | ডাটা স্টোরেজ |
| **DB Manager** | Mongo Express | 8081 | ডাটাবেস ম্যানেজমেন্ট টুল |

---

## 🏗️ আর্কিটেকচার

```
┌─────────────────────────────────────────────────────┐
│                   Internet                          │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────┐
│              AWS Security Group                     │
│  Port: 22, 80, 443, 5050, 5173, 8081              │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────┐
│           EC2 Instance (Ubuntu 22.04)              │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │       Docker & Docker Compose               │ │
│  │                                              │ │
│  │  ┌────────────┐  ┌────────────┐            │ │
│  │  │ Frontend   │  │ Backend    │            │ │
│  │  │ (Vite)     │  │ (Express)  │            │ │
│  │  │ :5173      │  │ :5050      │            │ │
│  │  └──────┬─────┘  └──────┬─────┘            │ │
│  │         │               │                  │ │
│  │         └───────┬───────┘                  │ │
│  │                 │                          │ │
│  │         ┌───────▼────────┐                │ │
│  │         │    MongoDB     │                │ │
│  │         │    :27017      │                │ │
│  │         └────────────────┘                │ │
│  │                                            │ │
│  │         ┌────────────────┐                │ │
│  │         │ Mongo Express  │                │ │
│  │         │ :8081          │                │ │
│  │         └────────────────┘                │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## ✅ প্রয়োজনীয়তা

### AWS Account:
- ✅ Active AWS Account
- ✅ IAM User সহ Access Key
- ✅ EC2 Launch করার অনুমতি

### Local Machine:
- ✅ SSH Client (Windows/Mac/Linux)
- ✅ ব্রাউজার (Chrome, Firefox, Safari, etc.)

### Basic Knowledge:
- ✅ Linux Terminal কমান্ড
- ✅ Docker ধারণা
- ✅ REST API বোঝা

---

## 🔧 AWS EC2 সেটআপ

### ধাপ ১: AWS Console-এ যাও

1. [AWS Console](https://console.aws.amazon.com) খোলো
2. Region বাছাই করো (উদাহরণ: `us-east-1`)

### ধাপ ২: EC2 Instance Launch করো

1. **Services** → **EC2**
2. **Instances** → **Launch Instances**
3. নিম্নলিখিত সেটিংস করো:

```
Name and tags: Ostad-Docker
AMI: Ubuntu Server 22.04 LTS
Instance type: t2.micro (Free Tier eligible) বা t3.small
Key pair: নতুন তৈরি করো (Save করো .pem ফাইল)
Network settings:
  - Create security group: চেক করো
  - Security group name: ostad-sg
Storage:
  - Size: 20 GB
  - gp3
```

4. **Launch Instance** ক্লিক করো

### ধাপ ৩: Instance চলার জন্য অপেক্ষা করো

Instance Status → **Running** না হওয়া পর্যন্ত অপেক্ষা করো।

### ধাপ ৪: Public IP এড্রেস নোট করো

```
Instances → তোমার Instance → Public IPv4 address
উদাহরণ: 13.51.255.162
```

---

## 🐳 Docker & Docker Compose ইনস্টল

### ধাপ ১: EC2-এ SSH কানেক্ট করো

```bash
ssh -i /path/to/your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

**উদাহরণ:**
```bash
ssh -i ~/Downloads/ostad-key.pem ubuntu@13.51.255.162
```

### ধাপ ২: System আপডেট করো

```bash
sudo apt update
sudo apt upgrade -y
```

### ধাপ ৩: Docker ইনস্টল করো

```bash
sudo apt install docker.io -y
sudo apt install docker-compose -y

# যদি docker-compose না থাকে:
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### ধাপ ৪: Git এবং Node.js ইনস্টল করো

```bash
sudo apt install git -y

curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y
```

### ধাপ ৫: Docker পার্মিশন সেট করো

```bash
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker-compose --version
```

---

## 📦 প্রজেক্ট ক্লোন এবং সেটআপ

### ধাপ ১: প্রজেক্ট ক্লোন করো

```bash
git clone https://github.com/sourabhossain/Ostad-Docker.git
cd Ostad-Docker
```

### ধাপ ২: ফোল্ডার স্ট্রাকচার চেক করো

```bash
ls -la
```

আপনি দেখবেন:
```
OstadServer/          # Backend (Express.js)
OstadUI/              # Frontend (React + Vite)
Dockerfile-server     # Backend Docker image
Dockerfile-UI         # Frontend Docker image
ostad.yaml            # Docker Compose file
```

### ধাপ ৩: Frontend .env তৈরি করো

```bash
cat > OstadUI/.env << 'EOF'
VITE_API_URL=http://YOUR_EC2_PUBLIC_IP:5050
EOF
```

**উদাহরণ:**
```bash
cat > OstadUI/.env << 'EOF'
VITE_API_URL=http://13.51.255.162:5050
EOF
```

### ধাপ ৪: Backend .env তৈরি করো (প্রয়োজনে)

```bash
cat > OstadServer/.env << 'EOF'
MONGO_URI=mongodb://ostad:ostad@mongo:27017/ostad
MONGO_USERNAME=ostad
MONGO_PASSWORD=ostad
PORT=5050
NODE_ENV=production
EOF
```

---

## 🔐 Security Group কনফিগারেশন

### ধাপ ১: AWS Console → Security Groups

1. EC2 Dashboard → **Security Groups**
2. তোমার **ostad-sg** বাছাই করো

### ধাপ ২: Inbound Rules যোগ করো

নিম্নলিখিত Rules যোগ করো (Edit Inbound rules):

| Type | Protocol | Port Range | Source | Description |
|------|----------|-----------|--------|-------------|
| SSH | TCP | 22 | 0.0.0.0/0 | Terminal Access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web Traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure Web |
| Custom TCP | TCP | 5050 | 0.0.0.0/0 | Backend API |
| Custom TCP | TCP | 5173 | 0.0.0.0/0 | Frontend |
| Custom TCP | TCP | 8081 | 0.0.0.0/0 | Mongo Express |

### ধাপ ৩: Save Rules

**Save rules** ক্লিক করো।

---

## 🚀 Docker Compose চালানো

### ধাপ ১: Compose ফাইল চেক করো

```bash
cat ostad.yaml
```

### ধাপ ২: All Containers চালাও

```bash
sudo docker-compose -f ostad.yaml up -d
```

**ফ্ল্যাগ ব্যাখ্যা:**
- `-f ostad.yaml` = ফাইল নাম
- `up` = সব containers চালাও
- `-d` = Detached mode (পটভূমিতে)

### ধাপ ৩: স্ট্যাটাস চেক করো

```bash
sudo docker-compose -f ostad.yaml ps
```

**সফল আউটপুট:**
```
NAME                 IMAGE                       STATUS
mongo                mongo:latest                Up 2 minutes
mongo-express        mongo-express               Up 1 minute
ostad-server         ostad-docker_ostad-server   Up 1 minute
ostad-ui             ostad-docker_ostad-ui       Up 30 seconds
```

সবগুলোতে **"Up X minutes"** থাকতে হবে।

### ধাপ ৪: লগ দেখো

```bash
# সব logs
sudo docker-compose -f ostad.yaml logs

# শুধু Backend
sudo docker-compose -f ostad.yaml logs ostad-server

# রিয়েল টাইমে লগ (Ctrl+C দিয়ে বন্ধ করো)
sudo docker-compose -f ostad.yaml logs -f
```

---

## 🧪 API টেস্টিং

### ধাপ ১: Get Students (সব ছাত্র দেখো)

```bash
curl -X GET http://YOUR_EC2_IP:5050/getStudents
```

**উদাহরণ:**
```bash
curl -X GET http://13.51.255.162:5050/getStudents
```

### ধাপ ২: Add Student (নতুন ছাত্র যোগ করো)

```bash
curl -X POST http://YOUR_EC2_IP:5050/addStudent \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ahmed Ali",
    "email": "ahmed@example.com",
    "dateOfBirth": "2000-05-15",
    "gender": "Male"
  }'
```

**সফল রেসপন্স:**
```json
{
  "message": "Student added successfully",
  "id": "6914e1b359214b44e14d1fa5"
}
```

### ধাপ ৩: ব্রাউজারে Frontend দেখো

ব্রাউজারে খোলো:
```
http://YOUR_EC2_IP:5173
```

**উদাহরণ:**
```
http://13.51.255.162:5173
```

---

## 💾 ডাটাবেস ম্যানেজমেন্ট

### Mongo Express দিয়ে দেখো

ব্রাউজারে খোলো:
```
http://YOUR_EC2_IP:8081
```

**লগইন করো:**
- Username: `admin`
- Password: `pass`

### MongoDB CLI দিয়ে দেখো

```bash
# MongoDB container-এ ঢুকো
sudo docker exec -it mongo /bin/bash

# MongoDB shell চালাও
mongosh

# MongoDB prompt এ:
use ostad
db.students.find()
```

### ডাটা Export করো

```bash
# JSON ফর্ম্যাটে export
sudo docker exec mongo mongodump --uri="mongodb://ostad:ostad@localhost:27017/ostad" --out=/backup/

# ডাউনলোড করো
sudo docker cp mongo:/backup ./backup
```

---

## 🔧 সমস্যা সমাধান

### সমস্যা ১: "docker: command not found"

**সমাধান:**
```bash
sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
```

### সমস্যা ২: "docker-compose: command not found"

**সমাধান:**
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### সমস্যা ৩: Port conflict

**লক্ষণ:** "address already in use"

**সমাধান:**
```bash
# পোর্ট ব্যবহারকারী খোজো
sudo lsof -i :5050

# Kill করো (অথবা নাম্বর বদলাও)
sudo kill -9 PID_NUMBER
```

### সমস্যা ৪: MongoDB কানেক্ট হচ্ছে না

**সমাধান:**
```bash
# MongoDB logs দেখো
sudo docker-compose -f ostad.yaml logs mongo

# MongoDB restart করো
sudo docker-compose -f ostad.yaml restart mongo

# অপেক্ষা করো ৩০ সেকেন্ড
sleep 30
```

### সমস্যা ৫: Frontend API call সমস্যা

**লক্ষণ:** Form submit হচ্ছে না, CORS error

**সমাধান:**
1. OstadUI/.env চেক করো - সঠিক API URL আছে?
2. Backend logs দেখো - কোনো error?
3. CORS headers আছে কিনা?

### সমস্যা ৬: ব্রাউজারে "Connection Timeout"

**সমাধান:**
1. Security Group-এ Port খোলা আছে কিনা চেক করো
2. `docker-compose ps` দিয়ে containers চলছে কিনা দেখো
3. ফায়ারওয়াল চেক করো

---

## ✅ চেকলিস্ট

সফল ডিপ্লয়মেন্টের জন্য এই লিস্ট চেক করো:

- [ ] AWS EC2 Instance চলছে
- [ ] SSH কানেক্ট করতে পারছ
- [ ] Docker ইনস্টল আছে (`docker --version`)
- [ ] Docker Compose ইনস্টল আছে (`docker-compose --version`)
- [ ] প্রজেক্ট ক্লোন করেছ
- [ ] Security Group Rules সব Port খোলা আছে
- [ ] OstadUI/.env সঠিক IP দিয়ে তৈরি
- [ ] `docker-compose up -d` সফল হয়েছে
- [ ] `docker-compose ps` সব "Up" দেখাচ্ছে
- [ ] Frontend ব্রাউজারে লোড হচ্ছে (`:5173`)
- [ ] Backend API কল করতে পারছ (`:5050`)
- [ ] Mongo Express লগইন করতে পারছ (`:8081`)
- [ ] নতুন ডাটা সেভ হচ্ছে (MongoDB এ যাচাই করেছ)
- [ ] ডাটা Frontend এ দেখা যাচ্ছে

---

## 🔄 রক্ষণাবেক্ষণ এবং আপডেট

### Daily Checks

```bash
# Containers স্ট্যাটাস
sudo docker-compose -f ostad.yaml ps

# যদি কিছু বন্ধ থাকে, আবার চালাও
sudo docker-compose -f ostad.yaml up -d
```

### Weekly Tasks

```bash
# System আপডেট
sudo apt update && sudo apt upgrade -y

# Docker logs সাফ করো
sudo docker system prune -a
```

### Database Backup

```bash
# Backup নিও
sudo docker exec mongo mongodump --uri="mongodb://ostad:ostad@localhost:27017/ostad" --out=/backup/$(date +%Y-%m-%d)

# Backup list দেখো
sudo docker exec mongo ls -la /backup/
```

### Code Update

```bash
# নতুন কোড pull করো
git pull origin main

# Rebuild containers
sudo docker-compose -f ostad.yaml down
sudo docker-compose -f ostad.yaml up -d --build
```

### Restart All

```bash
sudo docker-compose -f ostad.yaml restart
```

### Stop Everything

```bash
sudo docker-compose -f ostad.yaml down
```

---

## 📊 মনিটরিং

### Real-time Logs দেখো

```bash
sudo docker-compose -f ostad.yaml logs -f --tail=100
```

### Container Resource Usage

```bash
sudo docker stats
```

### Disk Space চেক করো

```bash
df -h
```

---

## 🔐 Security Best Practices

1. **SSH Key নিরাপদ রাখো**
   ```bash
   chmod 600 ~/your-key.pem
   ```

2. **Security Group শুধু প্রয়োজনীয় Port খোলো**
   - অপ্রয়োজনীয় Port বন্ধ রাখো

3. **MongoDB Password চেঞ্জ করো**
   ```bash
   # ostad.yaml এ change করো
   MONGO_INITDB_ROOT_PASSWORD: STRONG_PASSWORD_HERE
   ```

4. **Backend এ Authentication যোগ করো**
   - API এ JWT টোকেন implement করো

5. **HTTPS Enable করো**
   - AWS Certificate Manager দিয়ে SSL certificate যোগ করো

---

## 💰 খরচ অপটিমাইজেশন

| Instance | মাসিক খরচ | ব্যবহার |
|----------|-----------|--------|
| t2.micro | ≈ $10 | Free Tier (first 12 months) |
| t3.small | ≈ $20 | Light production |
| t3.medium | ≈ $40 | Medium production |

### খরচ কমানোর উপায়:
- Free Tier instance ব্যবহার করো (1 বছর)
- Stopped instance চার্জ হয় না (storage শুধু charge হয়)
- Reserved instances ব্যবহার করো (long-term)

---

## 🆘 জরুরি কমান্ড Reference

```bash
# সব logs একসাথে
sudo docker-compose -f ostad.yaml logs -f

# Container এ ঢুকো
sudo docker exec -it ostad-server /bin/bash

# MongoDB shell খোলো
sudo docker exec -it mongo mongosh -u ostad -p ostad

# সব কিছু বন্ধ করো (ডাটা রাখে)
sudo docker-compose -f ostad.yaml stop

# সব কিছু বন্ধ করো (ডাটা ডিলিট করে)
sudo docker-compose -f ostad.yaml down -v

# Specific container logs
sudo docker logs -f CONTAINER_ID

# System info
docker --version
docker-compose --version
uname -a
free -h
```

---

## 📚 Reference Links

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Express.js Documentation](https://expressjs.com/)
- [React + Vite Documentation](https://vitejs.dev/)

---

## 💬 সাপোর্ট এবং যোগাযোগ

- **GitHub Issues**: https://github.com/sourabhossain/Ostad-Docker/issues
- **Email**: sourabhossain@example.com

---

## 📝 পরিবর্তনের ইতিহাস

| সংস্করণ | তারিখ | পরিবর্তন |
|---------|------|---------|
| 1.0 | ২০২৫-১১-১২ | প্রাথমিক ডকুমেন্টেশন |

---

**শুভকামনা! 🎉 আপনার প্রজেক্ট সফলভাবে চলুক!**
