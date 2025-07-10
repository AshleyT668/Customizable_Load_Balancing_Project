# Load-Balancer-Distributed-System
## Project Overview
This project implements a lightweight, Dockerized load balancing system using Consistent Hashing to distribute client requests across multiple asynchronous server replicas. The system demonstrates core distributed systems concepts including virtual node mapping, health monitoring, fault recovery, container orchestration, and scalable routing—all simulated using local containers.

## Table of Contents
- [Installation and Setup](#installation-and-setup)
  - [Pre-requisites](#pre-requisites)
  - [Setup Instructions](#setup-instructions)
- [Basic Usage](#basic-usage)
  - [Running Docker Services](#running-docker-services)
  - [Running Python Files](#running-python-files)
- [Testing the Project](#testing-the-project)
  - [Endpoints](#endpoints)
  - [Testing with cURL](#testing-with-curl)
- [Load Balancer Performance Analysis](#load-balancer-performance-analysis)
  - [A-1: Requests Count Per Server Instance](#a-1-request-count-per-server-instance)
  - [A-2: Average Load of Servers for N=2 to 6](#a-2-average-load-of-servers-for-n--2-to-6)
  - [A-3: Failure Detection and Handling](#a-3-failure-detection-and-handling)
  - [A-4: Modifying the Hash Functions](#a-4-modifying-the-hash-functions)

## Installation and Setup
### Pre-requisites
- Python 3.9+
- Docker Version 20.10.23 or above
- Ubuntu 20.04 LTS or above

### Setup Instructions
- Clone the repository:
```bash
git clone https://github.com/AshleyT668/Customizable_Load_Balancing_Project.git 
```

- Navigate to the repository:
```bash
cd Customizable_Load_Balancing_Project
```

- Create a Python virtual environment:
```bash
python3 -m venv .venv
```

- Activate the virtual environment:
```bash
source .venv/bin/activate
```

- Install the required libraries:
```bash
pip install -r requirements.txt
```

## Basic Usage
### Running Docker Services
- Build images and start the services:
```bash
docker-compose up -d
```

- Verify that the containers are running:
```bash
docker-compose ps
```

- When you're done, you can stop and remove containers:
```bash
docker-compose down
```

### Running Python Files
- To run Python files while the virtual environment is active:
```bash
python app.py
```
## Testing the Project
Postman, command-line tools like `cURL`, or Python scripts using the `requests` module can be used to perform tests
for the server(s) and load balancer. 

### Endpoints
#### Server Endpoints
Each server should have the following endpoints:

| Endpoint     | Method | Description               |
| ------------ | ------ | ------------------------- |
| `/home`      | GET    | Returns a success message |
| `/heartbeat` | GET    | Used for health checks    |

#### Load Balancer Endpoints
The load balancer exposes the following endpoints:

| Endpoint  | Method | Description                                                                   |
|-----------| ------ |-------------------------------------------------------------------------------|
| `/add`    | POST   | Adds new backend servers                                                      |
| `/rm`     | DELETE | Removes existing backend servers                                              |
| `/rep`    | GET    | Lists the currently active backend servers                                    |
| `/<path>` | GET    | A catch-all route that handles other endpoints to be directed to the servers. |

- The load balancer is exposed to the public network through the host machine on port 5000.

### Testing with cURL
#### 1. Test Load Balancer Routing

```bash
curl http://localhost:5000/home
```

#### 2. Add a New Server
```bash
curl -X POST http://localhost:5000/add \
  -H "Content-Type: application/json" \
  -d '{"n": 1, "hostnames": ["server4:5000"]}'
```

#### 3. Remove a Server
```bash
curl -X DELETE http://localhost:5000/rm \
  -H "Content-Type: application/json" \
  -d '{"n": 1, "hostnames": ["server4:5000"]}'
```

#### 4. List Active Servers
```bash
curl http://localhost:5000/rep
```
## Load Balancer Performance Analysis

### A-1: Request Count Per Server Instance
- This experiment involved executing 10,000 async requests on the 3 server containers, with the results being displayed
in a bar chart.
- Asynchronous requests are launched by using `aiohttp` + `asyncio` for concurrent request handling.
![Test 1](https://github.com/user-attachments/assets/fdd56e1e-f5c6-4a77-a259-7e8db3bb4f34)


#### Observations
From the graph, most of the requests are handled by server2, followed by server1 and lastly server3.
Although quadratic probing improves clustering issues compared to linear probing, the distribution 
is still somewhat imbalanced, suggesting that the placement of virtual nodes is not entirely uniform
across the ring.

---

### A-2: Average Load of Servers for N = 2 to 6
- This experiment involved iteratively increasing the number of servers from 2 to 6 while launching async requests.
- The results are plotted on a line chart.
  
![Test 2](https://github.com/user-attachments/assets/544d4c5e-afbb-4ef8-aa18-c661f3adf9b1)


#### Observations
From the line chart, the average number of requests being handled per server is inversely proportional to the number of 
servers being handled by the load balancer. This behaviour is expected because more servers means fewer requests routed 
to a specific server, showing a case of horizontal scaling.

---

### A-3: Failure Detection and Handling
- This experiment analysed how the load balancer handles server failures.
- Server 1 was stopped to represent a server failure.

#### Observations
From the experiment, it was noted that majority of the requests were routed to server 2 compared to server 3, showing a 
significant imbalance which still existed based on the setup, despite the removal of server 1.

---

### A-4: Modifying the Hash Functions
- The hash ring functions were modified and results gathered, based on the change.
- These were the new hash functions (defined arbitrarily) for the requests and servers that were used:
  - Request Hash Function: $\quad H(i) = (3 \cdot i) \mod S$
  - Server Hash Function: $\quad \phi(i, j) = (i^2 + 4 \cdot j) \mod S$

Based on the results gathered, server 3 now handled more request traffic than the other servers, 
based on these new functions. Request distribution is still similar as it was before,with the only 
difference being the server handling the most requests.
