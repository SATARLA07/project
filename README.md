# project
ecommerce-chatbot/
├── backend/
│   ├── app.py
│   ├── chatbot.py
│   ├── utils.py
│   ├── requirements.txt
│   └── data/
│       ├── inventory_items.csv
│       ├── order_items.csv
│       ├── orders.csv
│       ├── products.csv
│       └── users.csv
├── frontend/
│   ├── package.json
│   └── src/
│       ├── App.js
│       ├── Chatbot.js
│       └── index.js
├── README.md


---

🧠 STEP-BY-STEP INSTRUCTIONS

🥇 STEP 1: Setup Project Locally

mkdir ecommerce-chatbot
cd ecommerce-chatbot
mkdir backend frontend


---

🥈 STEP 2: Download Dataset

Go to: https://github.com/recruit41/ecommerce-dataset
Download and place all .csv files in:

ecommerce-chatbot/backend/data/


---

🥉 STEP 3: Backend Code (/backend)

requirements.txt

flask
flask-cors
pandas


---

utils.py

import pandas as pd

inventory = pd.read_csv("data/inventory_items.csv")
orders = pd.read_csv("data/orders.csv")
order_items = pd.read_csv("data/order_items.csv")
products = pd.read_csv("data/products.csv")

def get_top_sold_products(n=5):
    top = order_items['product_id'].value_counts().head(n)
    result = pd.merge(top.rename("count").reset_index(), products, left_on='index', right_on='id')
    return result[['id', 'name', 'brand', 'count']].to_dict(orient='records')

def get_order_status(order_id):
    order = orders[orders['order_id'] == order_id]
    return order.iloc[0].to_dict() if not order.empty else {"error": "Order not found"}

def get_product_stock(product_name):
    matched = inventory[inventory['product_name'].str.lower() == product_name.lower()]
    in_stock = matched[matched['sold_at'].isna()]
    return {"product_name": product_name, "stock": len(in_stock)}


---

chatbot.py

from utils import get_top_sold_products, get_order_status, get_product_stock
import re

def respond_to_query(message):
    message = message.lower()

    if "top" in message and "sold" in message:
        return get_top_sold_products()

    elif "order" in message and "status" in message:
        match = re.search(r'\d+', message)
        if match:
            return get_order_status(int(match.group()))
        return {"error": "Please enter a valid order ID."}

    elif "how many" in message and '"' in message:
        match = re.findall(r'"([^"]+)"', message)
        if match:
            return get_product_stock(match[0])
        return {"error": "Product name must be in quotes."}

    return {"response": "I didn’t understand that. Try asking about orders, top products, or stock."}


---

app.py

from flask import Flask, request, jsonify
from flask_cors import CORS
from chatbot import respond_to_query

app = Flask(_name_)
CORS(app)

@app.route("/chat", methods=["POST"])
def chat():
    data = request.get_json()
    message = data.get("message", "")
    reply = respond_to_query(message)
    return jsonify(reply)

if _name_ == "_main_":
    app.run(debug=True)


---

🚀 Run Backend

cd backend
pip install -r requirements.txt
python app.py

Runs at: http://localhost:5000


---

🥇 STEP 4: Frontend Code (/frontend)

package.json

{
  "name": "chatbot-frontend",
  "version": "1.0.0",
  "dependencies": {
    "axios": "^1.6.2",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start"
  }
}


---

src/index.js

import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);


---

src/App.js

import React from "react";
import Chatbot from "./Chatbot";

function App() {
  return (
    <div style={{ padding: "20px" }}>
      <h2>Customer Support Chatbot</h2>
      <Chatbot />
    </div>
  );
}

export default App;


---

src/Chatbot.js

import React, { useState } from "react";
import axios from "axios";

function Chatbot() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");

  const sendMessage = async () => {
    if (!input) return;
    const userMsg = { sender: "You", text: input };
    setMessages([...messages, userMsg]);

    const res = await axios.post("http://localhost:5000/chat", { message: input });
    const botMsg = { sender: "Bot", text: JSON.stringify(res.data) };

    setMessages((msgs) => [...msgs, botMsg]);
    setInput("");
  };

  return (
    <div>
      <div style={{ border: "1px solid #ccc", padding: 10, height: 300, overflowY: "scroll" }}>
        {messages.map((msg, i) => (
          <div key={i}><strong>{msg.sender}:</strong> {msg.text}</div>
        ))}
      </div>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Ask a question..."
      />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
}

export default Chatbot;


---

🚀 Run Frontend

cd frontend
npm install
npm start

Runs at: http://localhost:3000


---

🟦 STEP 5: Upload to GitHub

1. Create GitHub repo:

Go to https://github.com → Create New Repository → Public → No README.

2. Upload Project:

cd ecommerce-chatbot
git init
git add .
git commit -m "Initial chatbot commit"
git remote add origin https://github.com/YOUR_USERNAME/ecommerce-chatbot.git
git branch -M main
git push -u origin main

> 🔁 Replace YOUR_USERNAME with your GitHub name.




---
