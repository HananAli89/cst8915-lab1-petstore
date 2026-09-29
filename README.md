# CST8915 Lab 1: Algonquin Pet Store on Azure VM

**Student Name**: Hanan Ali
**Student ID**: 041153451
**Course**: CST8915 Full-stack Cloud-native Development
**Semester**: Fall 2026

---

## Demo Video

🎥 [Watch Demo Video](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)

---

## Technical Explanations

### Order Service (Node.js)

The Order Service takes orders from the Store Front and sends them to RabbitMQ. It's built with Node.js and Express, using `amqplib` to connect to RabbitMQ and `cors` so the browser is allowed to call it. It has one endpoint, `POST /orders`, on port 3000. Node.js is a good fit here because the service mostly waits on network requests, and Node handles that well without blocking.

When an order comes in, the service connects to RabbitMQ, makes sure `order_queue` exists as a durable queue, and sends the order as a persistent message so it isn't lost if RabbitMQ restarts. It only replies "Order received" after RabbitMQ confirms it got the message; otherwise it returns an error. This separates taking an order from processing it, so a future service could read orders from the queue later. In this lab there's no consumer, which is why the message count kept going up.

### Product Service (Rust)

The Product Service provides the list of products. It's written in Rust using the Warp framework and the Tokio async runtime. It has one route, `GET /products`, that returns three products (Dog Food, Cat Food, Bird Seeds) as JSON, and it listens on `0.0.0.0:3030` so it can be reached from outside the VM. Rust was a good choice because it's fast and memory-safe, which matters for a service that's called every time the store page loads.

This service is independent. It doesn't need RabbitMQ or the Order Service to work, and it only talks to the Store Front, which requests the product list from it. It has CORS enabled for `GET` requests because the Store Front runs on a different port (8080), and the browser would block the request otherwise. The products are hardcoded for now, but since the service is separate, it could be switched to a database later without changing the other services.

### Store Front (Vue.js)

The Store Front is the website customers use to choose a product, enter a quantity, and place an order. It's built with Vue 3 and runs on port 8080. The main logic is in `OrderForm.vue`: it loads the products when the page opens, calculates the total automatically (2 × Dog Food showed $39.98 right away), and checks that a product is selected and the quantity is valid before sending the order. Vue works well for this because the page updates automatically whenever the data changes.

The Store Front connects to both backend services. It sends a `GET` request to the Product Service to load the products and a `POST` request to the Order Service to place the order. Because this code runs in the user's browser and not on the VM, both URLs had to point to the VM's public IP instead of `localhost`. The Store Front never talks to RabbitMQ directly, since that's the Order Service's job.

---

## Challenges and Learnings

- **VM size and region:** Standard B2s wasn't available on my Azure for Students subscription, so the VM was created as Standard_B2als_v2 in Sweden Central. It has the same 2 vCPUs and 4 GB RAM, so the lab worked the same.
- **SSH config permissions:** VS Code couldn't connect at first. The error showed that another Windows account on my laptop (`oracleadmin`, from my Oracle labs) had access to my SSH `config` file, and SSH refuses to use a config other users can access. I fixed it with `icacls` by removing inherited permissions and giving access only to my account.
- **Private key permissions:** After that, SSH gave `Load key ... Permission denied` because the key's permissions were granted to the wrong account name. Running `whoami` showed the exact account (`hanan\hanan`), and granting read access to that account fixed it. I learned that SSH on Windows is strict about file permissions, just like `chmod 400` on Linux.


---

## Acknowledgments

- Lab instructions and source code from the course repository: [26F_Lab1_CST8915](https://github.com/ramymohamed10/26F_Lab1_CST8915)
- ChatGPT (AI assistant) for troubleshooting help with SSH permissions on Windows.