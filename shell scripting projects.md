# Shell Scripting Projects

## Project 1: RAM Utilization Checker

To check the RAM utilization in the Linux machine and check whether it is reaching limit or not

```bash
#!/bin/bash

free_ram=$(free -mt | grep Total | awk '{print$4}')

thrushold=500

echo "$free_ram $thrushold"

if [[ $free_ram -le $thrushold ]]
then
        echo "under control"
else
        echo "limit crossed"
fi
```

## Project 2: Disk Utilization Alert

Checks all the useful disks and give alert if the disk utilization exceed 80% send a email notification || postfix needs to be installed

```bash
#!/bin/bash
EMAIL="santoshpalla27@gmail.com"
DISK_UTIL=$(df -kh | egrep -v "Filesystem |tmpfs" | awk '{print$5}' | cut -d "%" -f1)

for i in $DISK_UTIL
do
if [[ $i -gt 80 ]]
then
echo "disk limit crossed $i"  && echo "disk limit crossed $i"  | mail -s "limit excedded" $EMAIL
else
        echo "under control $i"
fi
done
```

### Continuous Version (using while loop)

```bash
#!/bin/bash

while true
do
EMAIL="santoshpalla27@gmail.com"
DISK_UTIL=$(df -kh | egrep -v "Filesystem |tmpfs" | awk '{print$5}' | cut -d "%" -f1)

for i in $DISK_UTIL
do
if [[ $i -gt 80 ]]
then
echo "disk limit crossed $i" | mail -s "limit excedded" $EMAIL
fi
done

sleep 9
done
```

## Project 3: File Archiver

In the given directory, if files more than given size 20mb or files older than 10 days,,compress those files and move to archive folder

```bash
cat real.sh
#!/bin/bash

base="/root/base"
archive="/root/archive"


if [ ! -d $base ]
then
        mkdir $base
fi

if [ ! -d $archive ]
then
        mkdir $archive
fi

for i in $(find /root/base -maxdepth 1 -iname "*" -type f  -user root -size -5M)
do
        gzip $i
        mv $i.gz $archive
done

echo "archive task" | mail -s "archive done please check"  santoshpalla27@gmail.com
```

## Archive and Copy to S3

```bash
#!/bin/bash

archive="/root/logs/"
s3_bucket="s3://archivess/archive/"  

logs=$(find /root/base -type f -iname "*" -size -1M)

for log in $logs
do
    gzip "$log"
    mv "${log}.gz" "$archive"

    aws s3 cp "$archive$(basename "${log}.gz")" "$s3_bucket" &> /dev/null
done
```

## MySQL Dump and Copy to S3

```bash
./sqldump.sh database-1.cnwcymm6otu4.us-east-1.rds.amazonaws.com  admin santoshpalla EmployeeDB
```

```bash
#!/bin/bash

hostname=$1
user=$2
password=$3
database=$4
date="backup-$(date "+%d-%m-%Y").sql"


mysqldump -h $hostname  -P 3306 -u $user -p$password $database > $date

if [ $? -eq 0 ]; then
    echo "Dump is a success"

    aws s3 cp /root/$date s3://database-log-sql/

    if [ $? -eq 0 ]; then
        echo "Copy success"
    else
        echo "Copy unsuccessful"
    fi
else
    echo "Dump unsuccessful"
fi
```