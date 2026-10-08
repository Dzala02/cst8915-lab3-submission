# CST8915 Lab 3: Algonquin Pet Store on Azure PaaS

**Student Name**: Dhruvansh Zala

**Student ID**: 041214130

**Course**: CST8915 Full-stack Cloud-native Development



---

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/pqAYnMV7W0M)

---

## My Service Repositories

- order-service: https://github.com/Dzala02/order-service
- product-service: https://github.com/Dzala02/product-service
- store-front: https://github.com/Dzala02/store-front

## How I Deployed Everything

- RabbitMQ is running on its own Azure VM (rabbitmq-vm).
- The order-service (Node.js) is running on Azure App Service.
- The product-service is running on Azure App Service. I rewrote it from Rust to Python (Flask) because Rust isn't supported on App Service. The old Rust code is still saved in the repo under the `lab2-rust` tag.
- The store-front is running on a VM (store-vm).

Note: I tried to create an Azure Static Web App for the store-front, but it failed with a policy error (RequestDisallowedByAzure) because of the new restriction on student accounts. Like the instructor said in the announcement, I deployed the store-front on a VM instead. It uses a .env file with the URLs of my two App Services.

---

## Reflection Questions

### 1. What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?

I couldn't actually use the GitHub Actions workflow for the store-front because Static Web Apps was blocked on my student account. Instead, I put the two URLs (VUE_APP_ORDER_SERVICE_URL and VUE_APP_PRODUCT_SERVICE_URL) in a .env file on the store-front VM. The tricky part was understanding that Vue uses these values when the app is built, not while it's running, because the final app is just JavaScript running in the browser. So if I change a URL, I have to restart or rebuild the app. I also learned not to put a slash at the end of the URLs, because the code already adds /products and /orders. I had a similar problem on the backend too. My RabbitMQ connection string showed up in the Azure portal, but it wasn't really saved, so the order-service tried to connect to localhost and failed. After I saved it properly and restarted the app, it worked.

### 2. How does deploying microservices on Azure Web App Service differ from running them locally?

When I ran the services on my VMs, I had to install everything myself (Node, Python, packages) and keep a terminal open for each service. With App Service, Azure takes care of the server and the runtime. I just connected my GitHub repo, and every time I push code, GitHub Actions builds and deploys it automatically. Some things were different from running locally though. Azure chooses the port and gives it to the app through the PORT variable, settings go in the portal instead of a .env file, and the Python app needed gunicorn as the startup command. Debugging is also different. My Python app first gave a 503 error, and I had to check the Log stream to see that the packages from requirements.txt were never installed. Running the deployment again fixed it.

### 3. Why is it important to use environment variables for configurations in a cloud environment?

In the cloud, the same code can run in many different places, and each place can have different settings like the RabbitMQ address or the API URLs. With environment variables, I can change these settings without touching the code. It's also safer, because my RabbitMQ password is only saved in the App Service settings and not in my GitHub repos. I saw this clearly in this lab. Between Lab 2 and Lab 3, all the addresses changed, but I only had to update the settings, not the code.

---

## Problems I Ran Into

- Static Web Apps was blocked by the student account policy, so I used a VM for the store-front.
- My Python product-service gave a 503 error at first because Azure didn't install the packages. Re-running the GitHub Actions deployment fixed it.
- The order-service couldn't connect to RabbitMQ (ECONNREFUSED 127.0.0.1) because the environment variable wasn't saved properly. Saving it again and restarting the app fixed it.
- I had to add all of the App Service's outbound IP addresses to the RabbitMQ VM's firewall rule for port 5672 so the order-service could connect.
- My first RabbitMQ password had an @ in it, which broke the connection string, so I changed it to letters and numbers only.
