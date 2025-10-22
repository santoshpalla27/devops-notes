# Shell Scripting Notes

## Pipeline and Grep Commands

- `|` will separate commands and used to use multiple commands and will take first command as input `|` is like and

- `grep` will take the word and give the line where the word is

- `egrep -v ""` will remove the word line which you define

- `grep -q 'some'` -- will check if some is exist and gives the exit status

- `if ls &> /dev/null` -- returns nothing

- `$?` is a special variable that holds exit status of last command if its success returns 0 or returns non zero like 0 if command returns nothing and something text then it will return non zero

- `command -v aws &> /dev/null` -- to check whether aws is present or not

- `> /dev/null` -- Discards the output (only return code matters for the if condition)

- Exit status work if command get successfull execution it will be true or if returns any error it will be false so we use --- `if command` instead of `[]`

- `-f` to check file exist and `-d` to check dir exist `!` is opposite

- `basename "${log}.gz"` -- this will remove the path of the file and only give sometimes useful in shell scripting

- `mv "${log}.gz" "$archive"` -- this `${}` is used when we add suffix or prefix to the variable and we use `""` so the file we are copying may have some spaces

- `echo 'net.ipv4.tcp_wmem = 4096 65536 134217728' | sudo tee -a /etc/sysctl.conf` -- to add data into a file

- `tr 'abc' 'xyz'` # Replaces 'a' with 'x', 'b' with 'y', and 'c' with 'z' -- tr used to replace also used for space

- `command | tee filename` -- used to add command output to the file

## Shell Script Basics

Every shell script has extension .sh and the file starts with /bin/bash which is also known as shebang

```bash
#!/bin/bash

content
```

## Running Shell Scripts

To run we first need to change the permission to executable can be done by below commands:

```bash
chmod 777 file

chmod +x file 

./file  # to run

# or

path/file
```

## Comments

- `#` single line comment

Multi line comment:

```bash
<<comment

csd
cds
cds

comment
```

## Variables

```bash
var_name=value      # no spaces

hostname=$(hostname)

# and used using $var_name

# can also given at runtime using 

f_name=$1

l_name=$2

./file Santosh reddy

# fixed variable

readonly f_name=$1
```

## Arrays

Arrays are space separated in shell scripting:

```bash
myarray=(1 2 3 4 "hello" "hi" )

# can be accessed as

echo ${myarray[0]

echo "${myarray[*]}"  # prints all the values

echo "${#myarray[*]}"  # prints the length of the array # is used

echo "${#myarray[*]:2:2}"  # "${#myarray[*]:2(index):2(no of values)}"   prints 2 values from 2nd index

myarray+=(new 1 2 )  # add new values to the array

# key-value pair in array

declare -A myarray

myarray=([name]=santosh [age]=22 [city]=hyderabad )

echo "${myarray[name]} ${myarray[age]} ${myarray[city]}"
```

## String Operations

```bash
string="hello how are you"

length=${#string}  # # is used to find the length of the variable

echo $length  # -- 19

${string^^}  # ^^ at the end to turn the var into uppercase

${string,,}  # ,, at the end to turn the var into lowercase

${string/hello/hi}  # to replace a word in the var we use var/old-word/new-word

${string:6:11}  # to slice the string var:staring-letter:no of letters from the starting letter
```

## User Input or Interaction

```bash
read var_name  # to store user input in the var

read -p "what is your name" name  # gives text on the user screen which is given in -p block
```

## Arithmetic Operations

```bash
a=3
b=4

let sum=$a+$b  # we use let to make it an arithmetic operations

let sum=$a*$b

# can also be performed using (())

echo " $(($a+$b))"
```

## Operators

- equal -- `-eq` / `==`
- greater than or equal -- `-ge`
- less than or equal -- `-le`
- not equal -- `-ne` / `!=`
- greater than -- `-gt`
- less than -- `-lt`

## Conditional Statements

We use `[]` only for valid condition like a string comparison or number check other wise we directly write the command like:

```bash
if  aws --version && ls
then
        echo "aws is installed"
else
        echo "aws is not installed"
fi
```

### If-Else

Make sure of spacing

Syntax:

```bash
if [ condition ]
then 
    echo " statement"
else
    echo "statement"
fi
```

Example:

```bash
read -p "enter the age" age

if [ $age -gt 18 ]
then
        echo "you can vote"
else
        echo " you cannot vote"
fi
```

### Elif

Syntax:

```bash
if [ condition ]
then 
    echo " statement "
elif [ condition ]
then
     echo " statement "
else 
    echo " statement "  
```

Example:

```bash
read -p "enter the age" age

if [ $age -gt 18 ]
then
        echo "you can vote"
elif [ $age -gt 16 ]
then
        echo " you can try "
else
        echo " you cannot vote"
fi
```

### Case

Choose between the options:

```bash
read -p "enter your choice" choice

case $choice in
        a) ls ;;  # to type multiple just click enter and end with ;;
        b) pwd ;;
        c) ls -lart ;;
        d) ps -ef ;;
        *) echo "please enter a valid input"
esac
```

## Logical Operators

- `condition && condition` -- if both conditions are true then true else false
- `condition || condition` -- if any condition is true then its true

You should use `[[ ]]` double for the logical conditions:

```bash
read age
if [[ $age -eq 18 || $age -ge 19 ]]
then
        echo "you are adult"
elif [[ $age -eq 17 && $age -le 17 ]]
then
        echo "you are not adult"
else
        echo "invalid input"
fi
```

## Loops

### For Loop

Example:

```bash
for i in 1 2 3 4 5 
do 
 echo "$i"
done
```

```bash
for i in {1..20}
do
        echo "$i"
done
```

Small script which prints all the data in the file:

names.txt:
```
santosh vamsy sai ram mouli
```

```bash
#!/bin/bash

file="./names.txt"

cat $file


for i in $(cat $file)
do
        echo "$i "

done
```

### While Loop

Will run until the condition is meet:

```bash
count=0
condition=10

while [[ $count -le $condition ]]
do
        echo "hello $count"
        let count++

done
```

### Infinite Loop

```bash
while true; 
do
    echo "This is an infinite loop."
    sleep 1  
done
```

## Functions

Syntax:

```bash
nameoffunction(){
 code
}
```

Example:

```bash
hello() {
echo "hello world"
}

hello
```

```bash
sum() {
num1=4
num2=3
let sum=$num1+$num2
echo "$sum"
}

sum
```

```bash
welcome() {
 echo "weclome $1"
 echo "you age is $2"
}

welcome santosh 22
```

## Arguments Passing in Shell Script

```bash
./script arg1 arg2 arg3

name=$1
age=$2
city=$3

echo " $name $age $city"

echo " all the arg are $@ "
echo " no of arg are $# "
```

## Break and Continue

```bash
num=5

for i in {1..10}
do
        if [[ $num -eq $i ]]
        then
                echo "num found "
                break
        fi
done

echo " $i"
```

Print all the odd numbers:

```bash
for i in 1 2 3 4 5 6 7 8 9 10
do
        let r=$i%2
        if [[ $r -eq 0 ]]
        then
                continue
        fi
        echo " odd number is $i "
done
```

## Important Commands

- `sleep` -- create a delay between two executions  ex: `sleep 1s`
- `exit` -- to stop script at a point
- `exit status $?` -- gives you status of previous command if that was a success

## File Redirection

- `ls > ls.txt` -- `>` is used to overrite the data into the given file and `>>` means append or add to the exisiting data

In case you don't want to print the output on terminal or write any file we can redirect the output to `/dev/null`:

Example:

```bash
#!/bin/bash

if ls &> /dev/null
then
        echo "fils exist"
else
        echo "files desnt exist"
fi
```

If condition will be true if ls works properly if it gets error prints else statement

- echo "print name of the pw script using ${0}"

To run a script in the background we use `command -- nohup ./script &` this will run the script in the background and will save output in nohup.out file

## Scheduling a Task

### At and Crontab

At time:
```
> path to the script
```

### Crontab

```bash
sudo yum install cronie -y
sudo systemctl start crond.service
sudo systemctl enable crond.service

crontab -l  # to list the cronjobs
crontab -e  # to edit

1 8 * * * /root/script.sh

# * * * * * path of the file
# | | | | | 
# | | | | ----- Day of the week (0 - 6) [Sunday = 0]
# | | | ------- Month (1 - 12)
# | | --------- Day of the month (1 - 31)
# | ----------- Hour (0 - 23)
# ------------- Minute (0 - 59)
```

## AWK Command

It is a text processing command

- `$` -- represents the column
- `$0` represents all data
- `NR` -- rows
- `NF` -- last column

```bash
awk '{print $2}' filename  # this is to print all lines 2nd column text $2 represents the exact num of coloum

awk '{print $2,$4}' filename  # this is to print all lines 2nd column text from multiple coloums

awk '{print $NF}' filename  # this is to print last coloum

awk '{print NF}' filename  # this will print the no of fields
```

Print every line which matches the word:

```bash
awk '/india/ {print $0}'
```

Can also use `""` for field separator

`-F` field separator like how the data is separated like using `,` or `:` we need to provide

Example data:

```
Name,Age,Occupation,Salary
John Doe,28,Engineer,75000
Jane Smith,34,Designer,65000
Alice Johnson,22,Intern,30000
Bob Brown,45,Manager,95000
Charlie Davis,30,Developer,80000
```

```bash
awk -F, '{print $3}' filename

awk -F, '{print $3}' data.csv
Occupation
Engineer
Designer
Intern
Manager
Developer

awk -F, '{if($2>32)  print$0}' data.csv      # checks the condition in second coloum and prints all the data which is above 32

Name,Age,Occupation,Salary
Jane Smith,34,Designer,65000
Bob Brown,45,Manager,95000

awk -F, 'NR=="4" {print $0}' data.csv      # prints NR row 4 and print0 everything

Alice Johnson,22,Intern,30000

awk -F, 'NR=="4",NR=="6"{print $0}' data.csv  # from 4th row to 6th row data 
```

We can also pass multiple delimeter using `awk -F[,:-] '{print$0}' file`

If we want to skip first row while filter we can use:

```
awk 'NR>1 {print$NF}'
```

## Cut Command

```bash
cut -c1 file  # prints all the first char of words

cut -c1,5 file  # prints all the first and fith char of words

cut -c1-5 file  # prints all the first to fith char of words

cut -d, -f1  # -d means what the text is separated with like , . : and -f coloum
```

## Sed Command

sed is a string editor

### Replace Text

```bash
sed -i 's/old/new/g' file.txt

sed -i 's/old/new/g' file.txt  # all occurrences

sed -i '3s/old/new/' file.txt  # only in specific line
```

### Delete a Specific Line

```bash
sed '5d' file.txt

sed '2,4d' file.txt  # 2 to 4
```

### Print

```bash
sed -n '3p' file.txt

sed -n '/pattern/p' file.txt
```

## Command Check

```bash
command -v name  # to check whether the app is installed or not

#!/bin/bash


if command -v aws  &> /dev/null
then
        echo "aws is installed"
else
        echo "aws is not installed"
fi
```

## Find Command

find command is used to search files in a dir hierarchy .

- `-maxdepth num` -- this will be the depth the hierarchy

```bash
find path -name "filename"  # to find a file with name

find path -iname "filename"  # to find a file with name which ignore uppercase or lower case

find path -size 50M  # search a file based on their size M for mb K for kb G for gb C for bytes

find path -type f  # to search for for files in the given dir or f for file and d for dire l for links etc

find path -user root  # search file for only given user

find path -perm 777 or /u=x  # find file on their permission

find path -iname a*  # find all files which name starts with letter a

find path -newer filename  # this finds files which are made after the creation of given file time

find path -empty  # give files which are empty in the given dir

find path -empty -exec rm {} \\;  # find and delets all the empty files in the dir

find path -size +1M -size -50M  # this finds files between 1 to 50mb in the given dir

find path -mtime 15  # this will find files which are modified in last 15 days

find path -name "*.log" -type f -delete  # to delete the files that are found
```

## Reading from a File

```bash
while read -r FILE; do
  ...
done < "$TMP_LIST"

# This line defines FILE as the variable to store each line read from the file $TMP_LIST.

# Each line (i.e., each S3 object key) is assigned to FILE for that loop iteration.
```