# Partitions 資料存儲邏輯



> **Before answering each question, let's add an overview of producer components:**

![overview of producer components](https://i.stack.imgur.com/qhGRl.png)

> #### 1. When a producer is producing a message - It will specify the topic it wants to send the message to, is that right? Does it care about partitions?

Producer will decide target partition to place any message, depending on:

* Partition id, if it's specified within the message
* **key % num partitions**, if no partition id is mentioned
* Round robin if neither **partition id** nor **message key** are available in message, meaning only value is available

