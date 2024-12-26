<template>
    <div>
      <h1>Decrypted Messages</h1>
      <ul>
        <li v-for="message in messages" :key="message.title">
          <strong>{{ message.title }}</strong>: {{ message.content }}
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  import forge from "node-forge";
  
  export default {
    data() {
      return {
        messages: [],
        encryptionKey: "3870wvK8ZqBGqEcbaMohvXHQnNiErY4QVeTChcZk2l4=", // Use the same key as in Django
      };
    },
    methods: {
      decryptData(encryptedBase64) {
        try {
          // Decode the Base64 string to get the IV and encrypted content
          const binaryData = forge.util.decode64(encryptedBase64);
          
          // Split the binary data into IV and encrypted data
          const iv = binaryData.slice(0, 16); // The first 16 bytes are the IV
          const encrypted = binaryData.slice(16); // The rest is the encrypted data
  
          // Convert the key from Base64 to a byte array
          const key = forge.util.decode64(this.encryptionKey); // This is the key for decryption
  
          // Create a cipher using AES-CBC mode
          const decipher = forge.cipher.createDecipher("AES-CBC", key);
          
          // Start the decryption process with the IV
          decipher.start({ iv: iv });
          
          // Update the decipher with the encrypted data
          decipher.update(forge.util.createBuffer(encrypted));
          
          // Finish the decryption
          const result = decipher.finish(); // `true` if decryption was successful
          
          if (result) {
            // If decryption was successful, get the decrypted text
            const decrypted = decipher.output.toString();  
            const cleanedText = decrypted.replace(/'/g, '"'); // Replace single quotes with double quotes

          // Try to parse the decrypted string as JSON
          try {
            const jsonData = JSON.parse(cleanedText); // Parse cleaned text as JSON
                return jsonData;  // Return the parsed JSON data
            } catch (e) {
                console.error("Decryption failed, invalid JSON format:", e);
                return null;
            }
            } else {
                console.error("Decryption failed.");
                return null;
            }
        } catch (error) {
            console.error("Decryption failed:", error);
            return null;
        }
      },
  
      async fetchMessages() {
        try {
          const response = await axios.get("http://127.0.0.1:8000/api/messages/");
          this.messages = this.decryptData(response.data.encrypted_data); // Decrypt and set the messages
          console.log(this.messages)
        } catch (error) {
          console.error("Error fetching messages:", error);
        }
      },
    },
    mounted() {
      this.fetchMessages();
    },
  };
  </script>
  