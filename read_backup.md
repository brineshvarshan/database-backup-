Automated Backup for MongoDB and PostgreSQL to Azure Blob Storage
Overview:
This project automates the backup of MongoDB and PostgreSQL databases and uploads them to Azure Blob Storage.

Prerequisites:

1.Azure Storage Account with a Blob Container.
2.MongoDB and PostgreSQL installed on your system.
3.Azure CLI installed and authenticated.
4.Cron Job setup for scheduling backups.

Implementation:
Step 1: Create an Azure Blob Storage Container

Step 2: Install PostgreSQL and MongoDB (if not installed)

For MongoDB:

sudo apt update
sudo apt install -y mongodb
sudo systemctl start mongodb
sudo systemctl enable mongodb
Verify Installation:

mongo --eval 'db.runCommand({ connectionStatus: 1 })'

For PostgreSQL:

sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
Verify Installation:

psql --version


Step 3: Configure Databases:

For MongoDB:
1.Inside PostgreSQL shell:

mongosh

2.Create a database (my_mongodb):

use my_mongodb
db.createCollection("users")
db.users.insertOne({name: "Alice"})

For PostgresSQL:
1.Inside PostgreSQL shell:

sudo -i -u postgres

2.Create PostgreSQL database:

CREATE DATABASE my_pgdb;


Step 4: Backup Script
1.Install dependencies:
pip install azure-storage-blob pymongo psycopg2-binary
2.Create backup_script.py:

import os
import subprocess
from datetime import datetime
from azure.storage.blob import BlobServiceClient
AZURE_STORAGE_ACCOUNT = os.getenv("AZURE_STORAGE_ACCOUNT")
AZURE_STORAGE_KEY = os.getenv("AZURE_STORAGE_KEY")
AZURE_CONTAINER = os.getenv("AZURE_CONTAINER")

BACKUP_DIR = "/home/user/backup"
TIMESTAMP = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
os.makedirs(BACKUP_DIR, exist_ok=True)
MONGO_DB = "my_mongodb"
mongo_backup_path = f"{BACKUP_DIR}/mongodb_backup_{TIMESTAMP}.gz"
subprocess.run(f"mongodump --db {MONGO_DB} --archive={mongo_backup_path} --gzip", shell=True)
PG_DB = "my_pgdb"
pg_backup_path = f"{BACKUP_DIR}/postgresql_backup_{TIMESTAMP}.dump"
subprocess.run(f"PGPASSWORD='bh' pg_dump -U postgres -d {PG_DB} -F c -f {pg_backup_path}", shell=True)
blob_service_client = BlobServiceClient(account_url=f"https://{AZURE_STORAGE_ACCOUNT}.blob.core.windows.net", credential=AZURE_STORAGE_KEY)
def upload_to_azure(file_path):
    blob_name = os.path.basename(file_path)
    blob_client = blob_service_client.get_blob_client(container=AZURE_CONTAINER, blob=blob_name)
    
    with open(file_path, "rb") as data:
        blob_client.upload_blob(data, overwrite=True)
    print(f"Uploaded: {blob_name}")
upload_to_azure(mongo_backup_path)
upload_to_azure(pg_backup_path)
subprocess.run(f"find {BACKUP_DIR} -type f -mtime +7 -delete", shell=True)
print("Backup & upload completed successfully!")


Step 5: Restore Commands:

Restore MongoDB Backup:

mongorestore --db your_mongo_db /home/user/backup/mongodb_backup_TIMESTAMP/


Restore PostgreSQL Backup:

PGPASSWORD='yourpassword' pg_restore -U postgres -d my_pgdb -F c /home/user/backup/postgresql_backup_TIMESTAMP.dump


Step 6: Automate Using Cron Job:

1.Edit the crontab:

crontab -e
2.Add the following line to run the backup daily at midnight:

0 2 * * * python3 /home/user/backup_script.py >> /home/user/backup/backup.log 2>&1



Conclusion:

This setup ensures that both MongoDB and PostgreSQL backups are taken regularly and securely stored in Azure Blob Storage. 🚀

_____________________________________________So the screenshots of this implementation are posted in the other files__________________________________________________________