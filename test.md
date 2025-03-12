# Terraform and Infrastructure Questions

## How can I ensure that Terraform does not provision resources without specific variables?
- Define variables without default values in `variables.tf`, so Terraform requires them at runtime.
- Use `TF_VAR_<var_name>` environment variables or pass `-var="var_name=value"` in CLI.
- Use `required_providers` and `precondition` blocks in Terraform 0.13+ to enforce constraints.

## How can I make sure the Terraform state file is backed up whenever I make changes or deploy new resources?
- Store the state in an **S3 backend with versioning** enabled to keep historical copies.
- Before applying changes, manually back up the state file:
  ```sh
  terraform state pull > backup.tfstate
  ```
- Enable **state locking** using **DynamoDB** (for S3 backend) to prevent simultaneous updates.

# Kubernetes Questions

## How do you troubleshoot or validate if a service can communicate with a pod in Kubernetes?
- Verify the service is correctly mapped to the pod:
  ```sh
  kubectl get svc -o wide
  ```
- Check if the pod is running and has the expected labels:
  ```sh
  kubectl get pods --show-labels
  ```
- Test connectivity using `curl` or `nc`:
  ```sh
  kubectl exec -it <pod> -- curl <service>:<port>
  ```
- Ensure there are no **NetworkPolicies** blocking communication:
  ```sh
  kubectl get networkpolicy
  ```

# AWS Questions

## How would you restrict access to an S3 bucket so that it is only accessible from a specific subnet?
- Use a **VPC Endpoint for S3**, ensuring that traffic does not go over the public internet.
- Apply a **bucket policy** that restricts access to a specific VPC or subnet:
  ```json
  {
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::your-bucket/*",
    "Condition": { "StringNotEquals": { "aws:SourceVpc": "vpc-xxxxxx" } }
  }
  ```
- Set `BlockPublicAccess` to `true` to prevent accidental public exposure.

## An EC2 instance is launched in a public subnet without a public IP. How can we install packages on this instance? What is the sequence of data flow, and which AWS networking components (NAT, IGW, etc.) would be involved?
- Since the instance lacks a public IP, it **cannot** directly access the internet.
- A **NAT Gateway** is required in a **public subnet** to route traffic via the **Internet Gateway (IGW)**.
- **Data flow:**
  ```
  EC2 (private IP) → Route Table (0.0.0.0/0) → NAT Gateway (public IP) → IGW → Internet
  ```

## Why can't we avoid using a NAT Gateway in the above scenario? What happens if we configure the route table to send traffic directly to the Internet Gateway (IGW)?
- **IGW only supports outbound traffic from instances with public IPs or Elastic IPs.**
- Since the EC2 instance only has a **private IP**, IGW **drops** the request.
- NAT Gateway is needed to **translate the private IP to a public IP** so that outbound internet access works.

# Security & Authentication Questions

## How do you configure passwordless authentication on a server?
- Generate an SSH key pair on the **client**:
  ```sh
  ssh-keygen -t rsa -b 4096
  ```
- Copy the **public key** to the **server**:
  ```sh
  ssh-copy-id user@server_ip
  ```
- Manually, you can append the key to `~/.ssh/authorized_keys` on the server.
- Ensure proper permissions:
  ```sh
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```
- If needed, disable password authentication by setting `PasswordAuthentication no` in `/etc/ssh/sshd_config`.

## Which SSH key should be kept on the client, and which should be stored on the server?
- **Client (local machine):** Stores the **private key** (`~/.ssh/id_rsa`). **Never share this key!**
- **Server (remote machine):** Stores the **public key** in `~/.ssh/authorized_keys`.
- Authentication works because the client signs a challenge using its private key, and the server verifies it using the public key.

## Is SSH key-based authentication symmetric or asymmetric encryption?
- **Asymmetric encryption** – Uses a **public-private key pair** for authentication.
- The **public key** encrypts the challenge, and only the **private key** can decrypt it.
- This ensures that even if someone intercepts the message, they **cannot** log in without the private key.

---

Would you like any modifications or additional explanations?
