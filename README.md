%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'16px'}}}%%
flowchart TB
    Users[("👥 Users<br/>Internet")]
    
    subgraph AWS ["☁️ AWS Cloud - us-east-1"]
        IGW["🚪 Internet Gateway"]
        
        subgraph VPC ["🏢 VPC: 10.10.0.0/16"]
            
            subgraph AZ1 ["📍 Availability Zone: us-east-1a"]
                subgraph PubSubnet1 ["🔴 Public Subnet: 10.10.1.0/24"]
                    NAT["🔀 NAT<br/>Gateway"]
                    ALB1["⚖️ Application<br/>Load Balancer"]
                end
                
                subgraph PrivSubnet1 ["🟢 Private Subnet: 10.10.3.0/24"]
                    Node1["🖥️ EKS Node 1<br/>t3.medium<br/>📦 Pods"]
                end
            end
            
            subgraph AZ2 ["📍 Availability Zone: us-east-1b"]
                subgraph PubSubnet2 ["🔴 Public Subnet: 10.10.2.0/24"]
                    ALB2["⚖️ ALB<br/>Endpoints"]
                end
                
                subgraph PrivSubnet2 ["🟢 Private Subnet: 10.10.4.0/24"]
                    Node2["🖥️ EKS Node 2<br/>t3.medium<br/>📦 Pods"]
                end
            end
            
            ControlPlane["☸️ EKS Control Plane<br/>ecommerce-eks-cluster<br/>Kubernetes v1.30<br/>━━━━━━━━━━<br/>📡 API Server<br/>⏰ Scheduler<br/>🎛️ Controller Manager"]
        end
        
        subgraph Security ["🔐 IAM & Security Layer"]
            OIDC["🔑 OIDC Provider<br/>IRSA Enabled"]
            Roles["👥 IAM Roles<br/>━━━━━━━━━━<br/>• ALB Controller Role<br/>• EBS CSI Driver Role<br/>• Node Group Role<br/>• Admin Access Entry"]
        end
        
        subgraph SystemApps ["🔧 System Components (kube-system)"]
            ALBCtrl["⚙️ AWS Load Balancer<br/>Controller v1.8.1"]
            EBSDriver["💾 EBS CSI Driver<br/>Persistent Storage"]
            DNS["🌐 CoreDNS"]
            Network["📡 VPC CNI + Kube-proxy"]
        end
        
        EBS[("💾 EBS Volumes<br/>gp2 Storage")]
    end
    
    Users -->|"HTTP/HTTPS"| IGW
    IGW --> ALB1 & ALB2
    ALB1 & ALB2 -->|"Route Traffic"| Node1 & Node2
    
    Node1 & Node2 -.->|"Pull Images<br/>API Calls"| NAT
    NAT -.-> IGW
    
    ControlPlane -.->|"Manages"| Node1 & Node2
    
    ALBCtrl -->|"Creates/Manages"| ALB1 & ALB2
    EBSDriver -->|"Provisions"| EBS
    EBS -.->|"Mounts to"| Node1 & Node2
    
    OIDC -.->|"Authenticates"| ALBCtrl & EBSDriver
    Roles -.->|"Permissions"| ALBCtrl & EBSDriver & Node1 & Node2
    
    SystemApps -.->|"Runs on"| Node1 & Node2
    
    style Users fill:#667eea,stroke:#764ba2,stroke-width:3px,color:#fff
    style AWS fill:#FF9900,stroke:#FF9900,stroke-width:3px
    style VPC fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style AZ1 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style AZ2 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style PubSubnet1 fill:#ffebee,stroke:#c62828,stroke-width:2px
    style PubSubnet2 fill:#ffebee,stroke:#c62828,stroke-width:2px
    style PrivSubnet1 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style PrivSubnet2 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style ControlPlane fill:#326ce5,stroke:#1565c0,stroke-width:3px,color:#fff
    style Security fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    style SystemApps fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    style Node1 fill:#66bb6a,stroke:#2e7d32,stroke-width:2px,color:#fff
    style Node2 fill:#66bb6a,stroke:#2e7d32,stroke-width:2px,color:#fff
    style EBS fill:#ffa726,stroke:#e65100,stroke-width:2px
