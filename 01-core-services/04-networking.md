# Networking

## The Plumbing

> 🖼️ **[IMAGE_PLACEHOLDER]** — VPC virtual private cloud public private subnets load balancer NAT

Networking connects everything. Understanding it prevents outages and security holes.

```
Internet
    │
    ▼
┌─────────────┐
│  Cloud DNS  │  (Route 53, Cloud DNS, Azure DNS)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  CDN Edge   │  (CloudFront, Cloud CDN, Azure CDN)
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────────┐
│                  VPC / VNet                  │
│  ┌──────────────┐    ┌──────────────┐       │
│  │ Public Subnet│    │Public Subnet │       │
│  │  Load Balancer│   │  NAT Gateway │       │
│  └──────┬───────┘    └──────┬───────┘       │
│         │                   │               │
│  ┌──────▼───────┐    ┌──────▼───────┐       │
│  │Private Subnet│    │Private Subnet│       │
│  │  Web Servers │    │  Database    │       │
│  └──────────────┘    └──────────────┘       │
└─────────────────────────────────────────────┘
```

## VPC / VNet

Your private network in the cloud. An isolated address space where your resources live.

```yaml
vpc:
  cidr: "10.0.0.0/16"           # 65,536 addresses
  subnets:
    - name: "public-a"
      cidr: "10.0.1.0/24"       # 256 addresses
      az: "us-east-1a"
      internet_access: true      # has Internet Gateway route
    - name: "private-a"
      cidr: "10.0.10.0/24"
      az: "us-east-1a"
      internet_access: false     # no direct internet route
    - name: "public-b"
      cidr: "10.0.2.0/24"
      az: "us-east-1b"
      internet_access: true
    - name: "private-b"
      cidr: "10.0.20.0/24"
      az: "us-east-1b"
      internet_access: false
```

**Rule:** Resources that receive traffic from the internet go in public subnets. Everything else goes in private subnets.

## Subnets

A subnet is a range of IP addresses within your VPC. Each subnet lives in one availability zone.

- **Public subnet** -- has a route to an Internet Gateway. Resources here can be reached from the internet.
- **Private subnet** -- no direct internet route. Resources here are unreachable from the internet. They access the internet via a NAT Gateway if needed.

## Security Groups

Virtual firewalls attached to resources. Stateful -- return traffic is automatically allowed.

```yaml
# Security group: web server
ingress:
  - port: 443
    source: "0.0.0.0/0"        # HTTPS from anywhere
  - port: 22
    source: "10.0.0.0/16"      # SSH from within VPC only

egress:
  - port: "all"
    destination: "0.0.0.0/0"   # Allow all outbound

# Security group: database
ingress:
  - port: 5432
    source: "sg-web-servers"   # Only from web server SG

egress: none                    # Database needs no outbound
```

**Principle:** Default deny. Open only what is needed. Reference other security groups instead of IP ranges when possible.

## Load Balancers

Distribute traffic across multiple targets. Health checks remove unhealthy instances from rotation.

```yaml
load_balancer:
  type: "application"           # Layer 7 (HTTP/HTTPS)
  scheme: "internet-facing"
  listeners:
    - port: 443
      protocol: "HTTPS"
      certificate: "arn:aws:acm:..."
      target_group:
        protocol: "HTTP"
        port: 8080
        health_check:
          path: "/health"
          interval: 30
          healthy_threshold: 2
          unhealthy_threshold: 3
```

Types: Application (HTTP, Layer 7), Network (TCP/UDP, Layer 4), Classic (legacy, avoid).

## DNS

Domain Name System. Translates `api.example.com` to a load balancer IP.

```yaml
dns:
  domain: "example.com"
  records:
    - name: "api.example.com"
      type: "A"
      alias_to: "load-balancer-dns-name"
    - name: "www.example.com"
      type: "CNAME"
      alias_to: "cdn-distribution-domain"
```

## Key Takeaway

VPC is your network. Subnets divide it. Security groups control access. Load balancers distribute traffic. DNS routes users. Get networking right and everything else is easier.
