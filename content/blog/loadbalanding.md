---
title: "Loadbalancing with Nginx."
description: "Step by step guide for setting up Nginx as a loadbalancer with Docker Compose."
dateString: July 2023
draft: flase
tags: ["Nginx", "Flask", "Load Balancing", "System Design", "Application Security"]
weight: 104
cover:
    image: "/blog/loadbalancing/cover.png"
---


Load balancing is like having multiple cash registers at a store. It ensures that when you have many customers (in this case, website visitors), no single register (server) gets overwhelmed. One of the most popular tools to handle this in the web world is Nginx. Let's break down what load balancing is, and how you can use it with a Flask app in a Dockerized environment.

## What is Load Balancing?

Load balancing is about spreading the load evenly across multiple servers. Imagine a bakery with many chefs baking cakes. Load balancing ensures that every chef gets an equal number of cake orders, making the process faster and more efficient. 

## Why Use Load Balancing?

Think about a popular website. Thousands of people visit it simultaneously. Without load balancing, one server would struggle to handle all these requests. Load balancing distributes these requests across multiple servers, improving performance, ensuring high availability, and preventing overloads.

## Nginx: Your Load Balancer

Nginx is a powerful, open-source web server that doubles as a fantastic load balancer. It stands in front of your web servers and directs incoming requests to the server best suited to handle them. Think of Nginx as the traffic cop at a busy intersection, guiding traffic smoothly.

## Load Balancing with Flask and Docker

Now, let's implement load balancing using Nginx with Flask, a Python web framework, all inside Docker containers. Here's how it works:

1. **Flask App**: We create a simple Flask app with just one endpoint that returns "Hello, World." Paste the below code in th `app.py` file.
	```py
	from  flask  import  Flask, jsonify
	app = Flask(__name__)
	@app.route('/')
	def  hello_world():
		response = {'message': 'Hello, World!'}
		return  jsonify(response)
	if  __name__ =='__main__':
		app.run(host='0.0.0.0', port=5000) 
	
	# add Flask==2.2.2 & Werkzeug==2.2.2 in requirements.txt

2. **Dockerizing Flask**: We use a Dockerfile to package our Flask app and its dependencies into a Docker image.
```dockerfile
FROM  python:3.8-slim
WORKDIR  /app
COPY  .  /app
RUN  pip  install  --trusted-host  pypi.python.org  -r  requirements.txt
EXPOSE  80
CMD  ["python",  "app.py"]
```

3. **Nginx Configuration**: We create an Nginx configuration file (`nginx.conf`) that defines how Nginx should distribute requests between multiple instances of our Flask app. It's like telling Nginx how to divide the cake orders among our chefs.
```nginx.conf
events {
	worker_connections 1024;
}
http {
	upstream backend {
		server backend-1:5000;
		server backend-2:5001;
	}
	server {
		listen 80;
		location / {
			proxy_pass http://backend;
		}
	}
}
```

4. **Docker Compose**: We use Docker Compose, a tool for defining and running multi-container Docker applications, to bring it all together. We define our Flask app and Nginx service in a `docker-compose.yml` file. Docker Compose starts multiple Flask app containers and an Nginx container, creating our load-balanced setup.
```docker-compose.yml
version: '3'
services:
  backend-1:
    build: .
    ports:
      - "5000"

  backend-2:
    build: .
    ports:
      - "5000"

  nginx:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "8080:80"

```
5. Now its time bring up the entire stack. Run the below command.
    ```sh
    docker-compose build && docker-compose up -d
    ```
6. Regularly monitor your setup to adjust resources as needed. Tools like Nginx logs and Docker stats are your friends here.

## Nginx for Security

Nginx offers more than just load balancing. It's also a powerful tool for enhancing the security of your web applications. Here are a few applications:

- **Reverse Proxy**: Nginx can act as a reverse proxy, sitting between your clients and your web application server. It can hide the details of your application's infrastructure, providing an extra layer of security.

- **SSL/TLS Termination**: Nginx can handle SSL/TLS encryption and decryption, offloading this task from your application servers. This not only improves security but also reduces the load on your application servers.

- **Access Control**: You can use Nginx to control access to your application based on IP addresses, HTTP headers, and more. This is particularly useful for protecting against malicious traffic.

- **Web Application Firewall (WAF)**: Nginx can be configured to filter and block malicious requests, protecting your application from common attacks like SQL injection and cross-site scripting (XSS).

- **DDoS Mitigation**: Nginx can help mitigate Distributed Denial of Service (DDoS) attacks by limiting the number of requests from a single IP or implementing rate limiting.


With Nginx and load balancing, your web application can handle high traffic loads with ease. It's like having an efficient system to serve cake orders without any chef👨‍🍳 getting overwhelmed. So, remember, when it comes to handling web traffic, think of Nginx as your friendly traffic cop👮 making sure everything runs smoothly. 

Happy load balancing ⚖️

