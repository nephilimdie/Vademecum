# Idempotence

---

## What is it?
**Idempotence** is a property of certain operations (in mathematics, computer science, networking, automation, etc.) where **executing them once or multiple times always leads to the same final result**.  

### Historical origin  
- The term comes from Latin *idem* (the same) + *potentia* (power): meaning **having the same effect**.  
- It was first introduced in **mathematics** in the 19th century, especially in the study of functions and set operations.  
- Later, it was applied in **computer science** and then in **distributed systems and APIs**.  

### Formal definition  
An operation `f(x)` is idempotent if:  

```
f(f(x)) = f(x)
```

Mathematical examples:  
- `abs(abs(x)) = abs(x)` (absolute value).  
- `min(min(a,b),c) = min(a,b,c)`.  

---

## How it works
- **Mathematics**: applying the function multiple times does not change the result.  
- **Computer science**: a command or request is idempotent if, even when repeated infinitely, it does not produce additional side effects.  

In **REST APIs**:  
- `GET` is idempotent: reading does not change the state.  
- `PUT` is idempotent: updating a resource with the same data leaves it unchanged.  
- `DELETE` is idempotent: deleting the same record multiple times does not cause errors.  
- `POST` is usually **not idempotent**: each call may create a new resource.  

---

## Applications
- **RESTful APIs**: ensures reliability in case of network retries.  
- **Distributed systems**: a duplicated message should not alter the system state.  
- **Databases**: commands like `UPSERT` or `INSERT IGNORE` reduce duplication issues.  
- **DevOps**: tools like **Ansible** or **Terraform** are designed to be idempotent: running the same playbook repeatedly results in the same final state.  
- **Digital payments**: systems try to enforce idempotence to avoid double charges (e.g., with unique request IDs).  

---

## Examples
### Mathematics
- `f(x) = |x|`  
- `f(x) = min(x, 10)`  

### REST APIs
- `PUT /users/123 { "name": "Stefano" }`  
   whether called once or 100 times, the user will always have `name=Stefano`.  

- `DELETE /users/123`  
  If the user no longer exists, nothing changes.  

### Databases
- `UPDATE accounts SET balance=100 WHERE id=1`: idempotent  
- `UPDATE accounts SET balance=balance+100 WHERE id=1`: non-idempotent  

### DevOps
```yaml
# Ansible playbook
- name: install nginx
  apt:
    name: nginx
    state: present
```
Running this 10 times will always result in Nginx installed **once**.  

### Programming (Python)
```python
def set_balance(account, value):
    account["balance"] = value   # idempotent
    return account

def add_balance(account, value):
    account["balance"] += value  # non-idempotent
    return account

acc = {"id": 1, "balance": 100}

print(set_balance(acc, 200))   # always results in 200
print(add_balance(acc, 200))   # changes every time
```

---

## Advantages and Limitations

**Advantages**:
- Safe retries.  
- Resilience in distributed systems.  
- Consistent and predictable state.  
- Simplified automation (e.g., DevOps).  

**Limitations**:
- Not all operations can be idempotent (e.g., financial transactions).  
- Requires careful design (duplicate handling, request IDs).  
- May reduce flexibility (some processes are inherently non-repeatable).  

---

## Evolutions
- **Cloud-native**: idempotence is a key principle for fault-tolerant systems.  
- **Event sourcing**: use of *idempotent consumers* that ignore duplicate events.  
- **Microservices**: deduplication mechanisms with unique request IDs.  
- **Blockchain**: transactions use nonce to prevent replay attacks and enforce idempotence.  

---

## Summary schema (Mind map)

```mermaid
mindmap
  root("Idempotence")
    What is it
      "Operation produces same result if repeated"
      "Origin: 19th century Mathematics"
    How it works
      "f(f(x)) = f(x)"
      "REST APIs: GET, PUT, DELETE are idempotent"
      "POST is not idempotent"
    Applications
      "REST APIs"
      "Distributed systems"
      "Databases"
      "DevOps (Ansible, Terraform)"
      "Digital payments"
    Examples
      "abs(x)"
      "PUT user"
      "DELETE user"
      "Fixed UPDATE"
    Advantages
      "Resilience"
      "Safe retries"
      "Automation"
    Limitations
      "Payments not idempotent"
      "Requires design"
    Evolutions
      "Cloud-native"
      "Event sourcing"
      "Microservices"
      "Blockchain"
```
