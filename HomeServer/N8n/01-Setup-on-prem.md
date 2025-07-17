<iframe title="How to Run n8n Locally (Full On-Premise Setup Tutorial)" src="https://www.youtube.com/embed/-ErfsM2TYsM?feature=oembed" height="113" width="200" allowfullscreen="" allow="fullscreen" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;"></iframe>


## Key Steps for Self-Hosting N8N

### Prerequisites for On-Premise Hosting

- **Domain Name**: You'll need to purchase a domain name, which can be a subdomain (e.g., `natn.hackwell.com`) [[01:17](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=77)].
    
- **Cloudflare**: Cloudflare Tunnels are recommended as a free solution to create a secure tunnel from your domain to your N8N instance, bypassing firewalls [[01:34](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=94)].
    

### Setting up Cloudflare

- **Account Creation**: Sign up for a free Cloudflare account [[02:08](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=128)].
    
- **Domain Registration/Connection**: Either connect an existing domain or register a new one through Cloudflare [[02:11](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=131)].
    
- **Zero Trust & Tunnels**: Navigate to the "Zero Trust" menu and then "Networks" to create a Cloudflare tunnel [[02:32](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=152)].
    
- **Tunnel Installation**: Install a small Cloudflare software piece on your Linux computer or server to establish the tunnel [[03:05](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=185)].
    
- **N8N Connection Setup**: Configure the tunnel to point to your N8N server's IP address, using HTTPS and setting TLS to "no TLS verify" [[04:06](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=246)].
    

### Installing N8N with Docker

- **SSH into Server**: Log into your server via SSH [[05:04](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=304)].
    
- **Install Docker**: Install Docker on your Ubuntu server using the APT repository [[05:16](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=316)].
    
- **Create N8N Directory**: Create a directory for N8N (e.g., `nhn-compose`) [[06:06](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=366)].
    
- **Create .env File**: Create and configure a `.env` file with your domain name, subdomain (N8N), timezone, and SSL email [[06:29](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=389)]. For local-only setups, you can use a made-up domain like `mycoolwebsite.local` and add it as a DNS entry on your local DNS server [[07:13](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=433)].
    
- **Create Local Files Directory**: Create a directory for local files [[08:29](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=509)].
    
- **Create docker-compose.yml File**: Create a `docker-compose.yml` file with the provided configuration, which includes both Trafik and N8N containers [[08:34](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=514)].
    
- **Spin Up Containers**: Run `docker compose up -d` to start the N8N and Trafik containers [[09:22](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=562)].
    
- **Verify Installation**: Use `docker ps` to confirm that the N8N and Trafik containers are running [[09:47](http://www.youtube.com/watch?v=-ErfsM2TYsM&t=587)].
    

---

## Setting Up Your First Automation (Workflow)

After installation, the presenter guides viewers through setting up their first automation, called a "workflow" in N8N. He explains basic N8N concepts, including:

- **Triggers**: How to initiate workflows, such as manually or on a schedule.
    
- **Nodes**: The building blocks of workflows, demonstrating how to add and connect them.
    
- **RSS Read Node**: Used to pull articles from RSS feeds, exemplified with Bleeping Computer and Krebs on Security.
    
- **Discord Node**: Used to send messages to Discord channels, including setting up webhooks and credentials.
    
- **Data Manipulation**: How N8N processes multiple items from a node and how to use JSON for data handling.
    
- **Limit Node**: To restrict the number of items passed between nodes.
    
- **Command Line Node**: To execute commands on the host system, demonstrating a ping command.
    
- **Merge Node**: To combine data from different branches of a workflow.
    
- **Pinning Data**: To preserve data during testing of individual nodes.
    
- **SSH Node**: Briefly mentioned as a way to execute commands on remote machines.
    

---

## Integrating AI with N8N

The video then delves into integrating AI with N8N:

- **Basic LLM Chain Node**: Used for AI tasks like summarizing articles.
    
- **Connecting AI Models**: Demonstrating connections to local models like Ollama and cloud models like OpenAI's ChatGPT.
    
- **Set Field Node**: To select and combine specific data fields for output.
    
- **Automating YouTube News**: Showing how to create an RSS feed for YouTube channels and filter videos by publication date.