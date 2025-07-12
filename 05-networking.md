# Networking Architecture

## Overview

This document outlines the comprehensive networking strategy for the AWS enterprise platform, covering internal and external communications, third-party integrations, on-premises connectivity, edge networking, security controls, and observability.

## Core Network Architecture

### 1. Multi-VPC Design Pattern

#### Hub and Spoke Architecture with Transit Gateway
```yaml
NetworkArchitecture:
  HubVPC:
    Name: "SharedServices-VPC"
    CIDR: "10.0.0.0/16"
    Purpose: "Centralized services (DNS, monitoring, security)"
    
  SpokeVPCs:
    Production:
      Name: "Production-VPC"
      CIDR: "10.1.0.0/16"
      Purpose: "Production workloads"
    
    NonProduction:
      Name: "NonProd-VPC"
      CIDR: "10.2.0.0/16"
      Purpose: "Development, staging, testing"
    
    Security:
      Name: "Security-VPC"
      CIDR: "10.3.0.0/16"
      Purpose: "Security tools, logging, SIEM"
    
    DMZ:
      Name: "DMZ-VPC"
      CIDR: "10.4.0.0/16"
      Purpose: "Public-facing services, WAF, load balancers"

TransitGateway:
  Name: "Platform-TGW"
  DefaultRouteTableAssociation: "disable"
  DefaultRouteTablePropagation: "disable"
  
  RouteTables:
    SharedServices:
      Name: "SharedServices-RT"
      Associations: ["SharedServices-VPC"]
      Propagations: ["Production-VPC", "NonProd-VPC", "Security-VPC"]
      
    Production:
      Name: "Production-RT"
      Associations: ["Production-VPC"]
      Propagations: ["SharedServices-VPC", "Security-VPC"]
      
    NonProduction:
      Name: "NonProd-RT"
      Associations: ["NonProd-VPC"]
      Propagations: ["SharedServices-VPC"]
      
    Security:
      Name: "Security-RT"
      Associations: ["Security-VPC"]
      Propagations: ["SharedServices-VPC", "Production-VPC", "NonProd-VPC"]
      
    DMZ:
      Name: "DMZ-RT"
      Associations: ["DMZ-VPC"]
      Propagations: ["Production-VPC"]
```

### 2. Subnet Strategy and IP Address Management

#### Hierarchical IP Addressing
```yaml
IPAddressStrategy:
  GlobalCIDR: "10.0.0.0/8"  # 16M addresses
  
  RegionalAllocation:
    USEast1: "10.1.0.0/12"   # 1M addresses
    USWest2: "10.16.0.0/12"  # 1M addresses
    EUWest1: "10.32.0.0/12"  # 1M addresses
    APSouth1: "10.48.0.0/12" # 1M addresses
  
  VPCAllocation:
    ProductionVPC:
      CIDR: "10.1.0.0/16"    # 65k addresses
      Subnets:
        PublicSubnets:
          AZ1: "10.1.0.0/20"  # 4k addresses
          AZ2: "10.1.16.0/20"
          AZ3: "10.1.32.0/20"
        PrivateSubnets:
          AZ1: "10.1.64.0/18" # 16k addresses
          AZ2: "10.1.128.0/18"
          AZ3: "10.1.192.0/18"
        DatabaseSubnets:
          AZ1: "10.1.48.0/24" # 256 addresses
          AZ2: "10.1.49.0/24"
          AZ3: "10.1.50.0/24"
        
IPAddressManagement:
  Tool: "AWS VPC IP Address Manager (IPAM)"
  Configuration:
    Pools:
      - Name: "Global-Pool"
        AddressFamilies: ["ipv4", "ipv6"]
        Locale: "us-east-1"
      - Name: "Regional-Pool-US"
        ParentPool: "Global-Pool"
        AddressFamilies: ["ipv4"]
        Locale: "us-east-1"
```

### 3. DNS Strategy

#### Route 53 Resolver and Private Hosted Zones
```yaml
DNSArchitecture:
  PublicHostedZones:
    - Name: "company.com"
      RecordTypes: ["A", "AAAA", "CNAME", "MX", "TXT"]
      HealthChecks: true
      GeolocationRouting: true
      
    - Name: "api.company.com"
      RecordTypes: ["A", "AAAA"]
      RoutingPolicy: "weighted"
      FailoverRouting: true
  
  PrivateHostedZones:
    - Name: "internal.company.com"
      VPCAssociations: ["Production-VPC", "SharedServices-VPC"]
      
    - Name: "eks.internal.company.com"
      VPCAssociations: ["Production-VPC"]
      Records:
        - Name: "*.eks.internal.company.com"
          Type: "A"
          Value: "${InternalLoadBalancer.IP}"

Route53Resolver:
  InboundEndpoints:
    - Name: "SharedServices-Inbound"
      VPC: "SharedServices-VPC"
      Subnets: ["private-subnet-1a", "private-subnet-1b"]
      SecurityGroup: "resolver-inbound-sg"
      
  OutboundEndpoints:
    - Name: "SharedServices-Outbound"
      VPC: "SharedServices-VPC"
      Subnets: ["private-subnet-1a", "private-subnet-1b"]
      SecurityGroup: "resolver-outbound-sg"
      
  ForwardingRules:
    - Name: "OnPremises-DNS"
      DomainName: "corp.company.com"
      TargetIPs: ["192.168.1.10", "192.168.1.11"]
      VPCAssociations: ["Production-VPC", "NonProd-VPC"]
```

## External Connectivity

### 1. Internet Gateway and NAT Gateway Strategy

#### High Availability NAT Gateway Design
```yaml
InternetConnectivity:
  InternetGateway:
    VPC: "DMZ-VPC"
    EnableDnsHostnames: true
    EnableDnsSupport: true
    
  NATGateways:
    # One per AZ for high availability
    AZ1:
      SubnetId: "public-subnet-1a"
      ElasticIP: "eip-nat-1a"
      
    AZ2:
      SubnetId: "public-subnet-1b"
      ElasticIP: "eip-nat-1b"
      
    AZ3:
      SubnetId: "public-subnet-1c"
      ElasticIP: "eip-nat-1c"

RouteTables:
  PublicRouteTable:
    Routes:
      - DestinationCidrBlock: "0.0.0.0/0"
        GatewayId: "${InternetGateway}"
        
  PrivateRouteTables:
    AZ1:
      Routes:
        - DestinationCidrBlock: "0.0.0.0/0"
          NatGatewayId: "${NATGateway-AZ1}"
        - DestinationCidrBlock: "10.0.0.0/8"
          TransitGatewayId: "${TransitGateway}"
```

### 2. Content Delivery and Edge Computing

#### CloudFront Distribution Strategy
```yaml
CloudFrontConfiguration:
  GlobalDistribution:
    PriceClass: "PriceClass_All"
    HttpVersion: "http2and3"
    IPV6Enabled: true
    
  Origins:
    APIOrigin:
      DomainName: "api-internal.company.com"
      CustomOriginConfig:
        HTTPPort: 80
        HTTPSPort: 443
        OriginProtocolPolicy: "https-only"
        OriginSSLProtocols: ["TLSv1.2", "TLSv1.3"]
        OriginReadTimeout: 30
        OriginKeepaliveTimeout: 5
        
    StaticContentOrigin:
      DomainName: "static-content.s3.amazonaws.com"
      S3OriginConfig:
        OriginAccessIdentity: "OAI-StaticContent"
        
  CacheBehaviors:
    - PathPattern: "/api/v1/*"
      TargetOriginId: "APIOrigin"
      ViewerProtocolPolicy: "redirect-to-https"
      AllowedMethods: ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
      CachedMethods: ["GET", "HEAD", "OPTIONS"]
      CachePolicyId: "CachingOptimizedForUncompressedObjects"
      TTL:
        DefaultTTL: 0
        MaxTTL: 31536000
        
    - PathPattern: "/static/*"
      TargetOriginId: "StaticContentOrigin"
      ViewerProtocolPolicy: "redirect-to-https"
      AllowedMethods: ["GET", "HEAD"]
      CachedMethods: ["GET", "HEAD"]
      CachePolicyId: "CachingOptimized"
      TTL:
        DefaultTTL: 86400
        MaxTTL: 31536000

EdgeLocations:
  LambdaAtEdge:
    - EventType: "viewer-request"
      FunctionName: "security-headers"
      Purpose: "Add security headers"
      
    - EventType: "origin-response"
      FunctionName: "cache-optimization"
      Purpose: "Optimize cache headers"
      
  CloudFrontFunctions:
    - Name: "url-rewrite"
      EventType: "viewer-request"
      Purpose: "URL normalization and routing"
```

### 3. API Gateway and External APIs

#### API Gateway Configuration
```yaml
APIGatewayConfiguration:
  Type: "HTTP"  # HTTP API for better performance and lower cost
  
  CustomDomainName:
    DomainName: "api.company.com"
    CertificateArn: "${ACMCertificate.Arn}"
    EndpointType: "REGIONAL"
    SecurityPolicy: "TLS_1_2"
    
  Throttling:
    BurstLimit: 5000
    RateLimit: 10000
    
  CORS:
    AllowOrigins: 
      - "https://app.company.com"
      - "https://admin.company.com"
    AllowMethods: ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    AllowHeaders: ["Content-Type", "Authorization", "X-API-Key"]
    MaxAge: 86400
    
  Authorizers:
    JWT:
      Name: "cognito-authorizer"
      Type: "JWT"
      JwtConfiguration:
        Issuer: "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_XXXXXXXXX"
        Audience: ["client-id-1", "client-id-2"]
      IdentitySource: "$request.header.Authorization"
      
  Routes:
    - Method: "ANY"
      Path: "/api/v1/{proxy+}"
      Integration:
        Type: "HTTP_PROXY"
        Uri: "http://internal-alb.internal.company.com/{proxy}"
        ConnectionType: "VPC_LINK"
        ConnectionId: "${VPCLink.Id}"
        
VPCLink:
  Name: "internal-alb-link"
  TargetArns: ["${InternalALB.Arn}"]
  SecurityGroupIds: ["${VPCLinkSecurityGroup.Id}"]
  SubnetIds: ["private-subnet-1a", "private-subnet-1b", "private-subnet-1c"]
```

## On-Premises and Hybrid Connectivity

### 1. AWS Direct Connect

#### Direct Connect Gateway Strategy
```yaml
DirectConnectConfiguration:
  VirtualInterfaces:
    Primary:
      VLAN: 100
      BGP_ASN: 65000
      ConnectionId: "dxcon-primary"
      Bandwidth: "10Gbps"
      Location: "Equinix DC2"
      
    Secondary:
      VLAN: 200
      BGP_ASN: 65000
      ConnectionId: "dxcon-secondary"
      Bandwidth: "10Gbps"
      Location: "Equinix DC6"
      
  DirectConnectGateway:
    Name: "Platform-DXGW"
    AmazonSideAsn: 64512
    
  VirtualPrivateGateways:
    Production:
      AmazonSideAsn: 64512
      VPCAttachment: "Production-VPC"
      
BGPConfiguration:
  OnPremisesASN: 65000
  AWSAdvertisedRoutes:
    - "10.0.0.0/8"   # AWS internal networks
    - "172.16.0.0/12" # Container networks
    
  OnPremisesAdvertisedRoutes:
    - "192.168.0.0/16" # Corporate networks
    - "172.20.0.0/16"  # Data center networks
```

### 2. Site-to-Site VPN Backup

#### VPN Gateway Configuration
```yaml
VPNConfiguration:
  CustomerGateway:
    - Name: "OnPrem-CGW-Primary"
      Type: "ipsec.1"
      PublicIP: "203.0.113.10"
      BGPAsn: 65000
      
    - Name: "OnPrem-CGW-Secondary"
      Type: "ipsec.1"
      PublicIP: "203.0.113.20"
      BGPAsn: 65000
      
  VPNConnections:
    Primary:
      CustomerGatewayId: "${CustomerGateway-Primary}"
      Type: "ipsec.1"
      StaticRoutesOnly: false
      Tunnels:
        - PreSharedKey: "${SecretManager.VPN-PSK-1}"
        - PreSharedKey: "${SecretManager.VPN-PSK-2}"
          
  TransitGatewayAttachment:
    VPNConnectionId: "${VPNConnection-Primary}"
    TransitGatewayId: "${TransitGateway}"
```

### 3. SD-WAN Integration

#### Third-Party SD-WAN Integration
```yaml
SDWANIntegration:
  Providers: ["Cisco", "VMware VeloCloud", "Silver Peak"]
  
  Architecture:
    HubSites:
      - Location: "AWS us-east-1"
        DeviceType: "Virtual"
        Bandwidth: "1Gbps"
        Redundancy: "Active-Active"
        
    BranchSites:
      - Location: "Office-NYC"
        DeviceType: "Physical"
        Bandwidth: "100Mbps"
        Redundancy: "Active-Standby"
        
  Policies:
    TrafficSteering:
      CriticalApps: "Direct Connect"
      GeneralTraffic: "Internet + VPN"
      BackupPath: "Site-to-Site VPN"
      
    QoS:
      Voice: "Priority 1"
      Video: "Priority 2"
      CriticalData: "Priority 3"
      BestEffort: "Priority 4"
```

## Multi-Cloud and Partner Connectivity

### 1. Multi-Cloud Networking

#### Azure and GCP Connectivity
```yaml
MultiCloudConnectivity:
  Azure:
    ExpressRoute:
      CircuitSKU: "Standard"
      Bandwidth: "1Gbps"
      ServiceProvider: "Equinix"
      PeeringLocation: "Washington DC"
      
    VirtualNetworkGateway:
      Type: "ExpressRoute"
      SKU: "HighPerformance"
      VirtualNetwork: "Hub-VNet"
      
  GCP:
    CloudInterconnect:
      Type: "Dedicated"
      Bandwidth: "1Gbps"
      Location: "Equinix DC2"
      
    CloudRouter:
      Name: "aws-interconnect-router"
      ASN: 65001
      
  CrossCloudRouting:
    HubAndSpoke:
      Hub: "AWS"
      Spokes: ["Azure", "GCP"]
      RoutingProtocol: "BGP"
```

### 2. Partner and Third-Party Integrations

#### Secure API Connectivity
```yaml
PartnerConnectivity:
  APIGatewayIntegrations:
    PartnerA:
      Type: "HTTP_PROXY"
      EndpointURL: "https://api.partner-a.com"
      Authentication: "API_KEY"
      RateLimiting:
        RequestsPerSecond: 100
        BurstCapacity: 200
        
    PartnerB:
      Type: "VPC_LINK"
      EndpointURL: "https://private-api.partner-b.com"
      Authentication: "MUTUAL_TLS"
      
  PrivateLink:
    Providers:
      - Name: "Snowflake"
        ServiceName: "com.amazonaws.vpce.us-east-1.vpce-svc-xxxxx"
        
      - Name: "Databricks"
        ServiceName: "com.amazonaws.vpce.us-east-1.vpce-svc-yyyyy"
        
  NetworkFirewall:
    Rules:
      - Name: "allow-partner-apis"
        Priority: 100
        Action: "ALLOW"
        Protocol: "HTTPS"
        Destination: "api.trusted-partner.com"
```

## Service Mesh and Internal Networking

### 1. Istio Service Mesh

#### Service Mesh Configuration
```yaml
IstioConfiguration:
  ControlPlane:
    Version: "1.20.0"
    Components:
      Pilot:
        Resources:
          Requests:
            CPU: "500m"
            Memory: "2Gi"
          Limits:
            CPU: "1"
            Memory: "4Gi"
            
  DataPlane:
    SidecarInjection:
      Policy: "enabled"
      Template: "sidecar"
      
  Gateways:
    IngressGateway:
      Type: "LoadBalancer"
      Service: "istio-ingressgateway"
      Ports:
        - Port: 80
          Protocol: "HTTP"
        - Port: 443
          Protocol: "HTTPS"
          
    EgressGateway:
      Type: "ClusterIP"
      Service: "istio-egressgateway"
      
  VirtualServices:
    - Name: "api-service"
      Hosts: ["api.internal.company.com"]
      HTTP:
        - Match:
            - Uri:
                Prefix: "/v1/"
          Route:
            - Destination:
                Host: "api-service.production.svc.cluster.local"
                Port:
                  Number: 8080
              Weight: 90
            - Destination:
                Host: "api-service-canary.production.svc.cluster.local"
                Port:
                  Number: 8080
              Weight: 10
```

### 2. Network Policies and Microsegmentation

#### Kubernetes Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-tier-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
    - podSelector:
        matchLabels:
          app: istio-proxy
    ports:
    - protocol: TCP
      port: 8080
      
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: api
    ports:
    - protocol: TCP
      port: 8080
      
  - to: []  # Allow all DNS
    ports:
    - protocol: UDP
      port: 53
      
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-tier-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: api
    ports:
    - protocol: TCP
      port: 5432
```

## Security and Network Controls

### 1. AWS Network Firewall

#### Stateful and Stateless Rules
```yaml
NetworkFirewall:
  FirewallPolicy:
    StatelessDefaultActions:
      - "aws:pass"
    StatefulDefaultActions:
      - "aws:drop_strict"
      
    StatelessRuleGroups:
      - Name: "block-malicious-ips"
        Priority: 100
        Rules:
          - RuleDefinition:
              MatchAttributes:
                Sources: ["198.51.100.0/24"]
              Actions: ["aws:drop"]
              
    StatefulRuleGroups:
      - Name: "allow-https-outbound"
        Rules:
          - Header:
              Protocol: "TCP"
              Source: "10.0.0.0/8"
              SourcePort: "ANY"
              Direction: "FORWARD"
              Destination: "ANY"
              DestinationPort: "443"
            Options:
              - Keyword: "sid"
                Settings: ["1"]
            Action: "PASS"
            
  Suricata Rules: |
    alert http any any -> any any (msg:"Malicious User Agent"; 
    http.user_agent; content:"BadBot"; sid:1000001; rev:1;)
    
    pass tcp 10.0.0.0/8 any -> any 443 (msg:"Allow HTTPS"; sid:1000002; rev:1;)
    
    drop tcp any any -> 10.0.0.0/8 22 (msg:"Block SSH from Internet"; sid:1000003; rev:1;)
```

### 2. VPC Flow Logs and Network Monitoring

#### Comprehensive Flow Logging
```yaml
VPCFlowLogs:
  Configuration:
    ResourceType: "VPC"
    TrafficType: "ALL"
    LogFormat: "${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${windowstart} ${windowend} ${action} ${flowlogstatus}"
    
  Destinations:
    CloudWatchLogs:
      LogGroupName: "/vpc/flowlogs"
      RetentionInDays: 30
      
    S3:
      BucketName: "platform-flowlogs"
      Prefix: "year=${year}/month=${month}/day=${day}/hour=${hour}/"
      Compression: "GZIP"
      
  Analysis:
    VPCFlowLogsInsights:
      Queries:
        - Name: "Top Talkers"
          Query: |
            fields @timestamp, srcaddr, dstaddr, bytes
            | filter action = "ACCEPT"
            | stats sum(bytes) as total_bytes by srcaddr
            | sort total_bytes desc
            | limit 20
            
        - Name: "Rejected Traffic"
          Query: |
            fields @timestamp, srcaddr, dstaddr, srcport, dstport, protocol
            | filter action = "REJECT"
            | stats count() as reject_count by srcaddr, dstaddr, dstport
            | sort reject_count desc
```

## Network Observability and Monitoring

### 1. Network Performance Monitoring

#### CloudWatch Network Insights
```yaml
NetworkMonitoring:
  VPCReachabilityAnalyzer:
    Paths:
      - Name: "web-to-database"
        Source: "eni-web-server"
        Destination: "eni-database-server"
        Protocol: "tcp"
        DestinationPort: 5432
        
  NetworkInsights:
    PathAnalysis:
      - Source: "igw-12345678"
        Destination: "i-abcdef123456"
        Protocol: "tcp"
        Port: 80
        
  TransitGatewayNetworkInsights:
    RouteAnalysis:
      - TransitGatewayRouteTableId: "tgw-rtb-12345678"
        DestinationCidrBlock: "10.1.0.0/16"
        
CustomMetrics:
  NetworkLatency:
    - MetricName: "NetworkLatency"
      Namespace: "Platform/Networking"
      Dimensions:
        Source: "us-east-1a"
        Destination: "us-west-2a"
      Unit: "Milliseconds"
      
  Throughput:
    - MetricName: "NetworkThroughput"
      Namespace: "Platform/Networking"
      Dimensions:
        Interface: "eni-12345678"
        Direction: "In"
      Unit: "Bytes/Second"
```

### 2. Distributed Tracing for Network Calls

#### X-Ray Network Tracing
```yaml
XRayNetworkTracing:
  ServiceMap:
    Services:
      - Name: "api-gateway"
        Type: "AWS::ApiGateway"
        
      - Name: "load-balancer"
        Type: "AWS::ApplicationLoadBalancer"
        
      - Name: "kubernetes-service"
        Type: "AWS::EKS::Service"
        
  TraceSegments:
    HTTPCalls:
      - URL: "https://api.company.com/v1/users"
        Method: "GET"
        Duration: 150
        StatusCode: 200
        
    DatabaseCalls:
      - ConnectionString: "postgresql://db.internal.company.com:5432/app"
        Query: "SELECT * FROM users WHERE id = $1"
        Duration: 25
        
  Annotations:
    - Key: "region"
      Value: "us-east-1"
    - Key: "availability_zone"
      Value: "us-east-1a"
    - Key: "network_path"
      Value: "external->alb->eks->rds"
```

This comprehensive networking architecture provides secure, scalable, and observable connectivity for all platform components while maintaining flexibility for future growth and integration requirements.
