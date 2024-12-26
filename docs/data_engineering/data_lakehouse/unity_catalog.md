# 🗂️ Unity Catalog

## Getting started (Docker)

1. Clone repository:
   ```sh
   git clone https://github.com/unitycatalog/unitycatalog/tree/main
   ```
2. Start server:
   ```sh
   docker build -t unity_catalog_server .
   docker run -it -p 8080:8080 unity_catalog_server
   ```
3. Start UI:
   ```sh
   docker build -t unity_catalog_ui --build-arg PROXY_HOST=YOUR_IP_ADDRESS ui/.
   docker run -it -p 3000:3000 unity_catalog_ui
   ```
