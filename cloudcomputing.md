# Cloud comuting Notes -02-10-2025
# 1  What is Cloud Computing?

Cloud computing is the delivery of computing services (like servers, storage, databases, networking, software, analytics, and intelligence) over the internet ("the cloud") instead of relying on local servers or personal computers.

Pay-as-you-go model (you pay only for what you use).

Provides scalability, flexibility, and cost efficiency.

Examples: AWS, Microsoft Azure, Google Cloud Platform (GCP).

# 2 Traditional Computing

On-premises model: All hardware, software, storage, and networking are managed in-house.

Requires physical servers and data centers.

High upfront cost for buying infrastructure.

Limited scalability (difficult to quickly expand resources).

Full responsibility on company for maintenance, updates, and security.

# 3 Modern Cloud Computing

Infrastructure is hosted on cloud providers (AWS, Azure, GCP).

On-demand resources (scale up or down anytime).

Low upfront cost, mostly operational expenses (OPEX).

Maintenance, upgrades, and security largely handled by cloud vendor.

Access from anywhere with internet.

Supports automation, DevOps, and AI/ML integration.

# 4 Cloud Economics
1) -free tier 2 - ondemand model (pay per use pay for the size of the thing you asked for) 3 - reservation models 4 - Volume discounts

# 5 Cloud Design Principals

1) design for failures -be intentional about failures add redudancy  
2) decouple components - tight coupling - losely coupled with queue put queue are scaling layers in between components
3) Impletement elasticity 
4) think parallel