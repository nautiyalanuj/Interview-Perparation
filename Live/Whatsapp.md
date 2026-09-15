# Browser Testing
- While testing I have found that whatsapp uses websocket for communication in browser.
- We are creating one connection and for all the chats in different group/individual chat we are sending the message using that websocket only.
- While uploading images we are uploading same to the blob and looks like while sending same we are passing link to same.
- There was no message visible may be due to encryption in websocket.
<img width="1880" height="987" alt="image" src="https://github.com/user-attachments/assets/d6d66b14-ed63-4a72-a3d9-9722d3776213" />
