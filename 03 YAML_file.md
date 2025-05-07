# **YAML Essentials**

YAML (YAML Ain't Markup Language) is a human-friendly data serialization standard used for configuration files (like Ansible, Docker, Kubernetes) and data exchange.

## **1. Basic YAML Syntax**

### **Key Features**
- Uses **indentation** (spaces, no tabs)
- Case **sensitive**
- Supports **comments** (`#`)
- File extension: `.yaml` or `.yml`

### **Basic Structure**
```yaml
# Key-value pairs
name: John Doe
age: 30
is_active: true

# List/array
fruits:
  - Apple
  - Banana
  - Orange

# Nested structure
address:
  street: 123 Main St
  city: New York
  zip: 10001
```

## **2. Data Types**

| Type | Example | Notes |
|------|---------|-------|
| **String** | `name: "John"` | Quotes optional |
| **Number** | `age: 25` | Integer/float |
| **Boolean** | `active: true` | `true`/`false` |
| **Null** | `middle_name: null` | `null`/`~` |
| **Timestamp** | `created: 2023-05-15` | ISO 8601 format |

## **3. Advanced Structures**

### **Multi-line Strings**
```yaml
description: |
  This is a multi-line
  string that preserves
  line breaks.

summary: >
  This folds newlines
  into spaces for a
  single paragraph.
```

### **Anchors & Aliases (Reuse)**
```yaml
defaults: &defaults
  timeout: 30
  retries: 3

server:
  <<: *defaults
  host: example.com
```

### **Complex Nesting**
```yaml
employees:
  - name: Alice
    skills:
      primary: Java
      secondary: Python
  - name: Bob
    skills:
      primary: JavaScript
      secondary: SQL
```

## **4. YAML in Configuration Files**

### **Ansible Example**
```yaml
---
- name: Install packages
  hosts: all
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"
```

### **Docker Compose Example**
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: example
```

## **5. Validation & Tools**

### **Online Validators**
- [YAML Lint](https://www.yamllint.com/)
- [JSON to YAML Converter](https://www.json2yaml.com/)

### **Command Line Tools**
```bash
# Install yamllint
pip install yamllint

# Validate file
yamllint config.yml

# Convert JSON to YAML
python -c 'import sys, yaml, json; print(yaml.dump(json.loads(sys.stdin.read())))' < file.json
```

## **6. Common Pitfalls**

### **Indentation Errors**
❌ Wrong:
```yaml
server:
config:  # Missing indentation
  port: 80
```

✅ Correct:
```yaml
server:
  config:
    port: 80
```

### **Boolean Interpretation**
❌ Wrong (becomes string):
```yaml
enabled: "yes"  # String, not boolean
```

✅ Correct:
```yaml
enabled: yes  # Boolean
# Or explicitly:
enabled: true
```

## **7. YAML vs JSON**

| Feature | YAML | JSON |
|---------|------|------|
| **Readability** | High | Low |
| **Comments** | Yes | No |
| **Multi-line** | Yes | No |
| **Speed** | Slower | Faster |
| **Usage** | Configs | APIs |

## **8. Best Practices**
1. **Use 2-space indentation** (no tabs)
2. **Quote strings** with special chars (`:`, `{`, `[`)
3. **Start documents** with `---`
4. **End documents** with `...` (optional)
5. **Use `.yml` extension** for compatibility

## **9. Real-World Examples**

### **Kubernetes Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### **GitHub Actions Workflow**
```yaml
name: CI Pipeline
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run tests
      run: pytest
```
