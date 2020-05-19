---
date: 2019-03-30
title: Cheatsheet for Shell Programming
tags: ["Tools"]
categories: ["DevOps"]
---

## Variables

```shell
NAME="John"
echo $NAME
echo "$NAME"
echo "${NAME}!"
```

## String

```shell
NAME="John"
echo "Hi $NAME"  #=> Hi John
echo 'Hi $NAME'  #=> Hi $NAME
```

## Array

```shell
Fruits=('Apple' 'Banana' 'Orange')
Fruits[0]="Apple"
Fruits[1]="Banana"
Fruits[2]="Orange"
```



## And / Or

```shell
git commit && git push
git commit || echo "Commit failed"
```

## Function

```shell
get_name() {
  echo "John"
}

echo "You are $(get_name)"

myfunc() {
    echo "hello $1"
}
# Same as above (alternate syntax)
function myfunc() {
    echo "hello $1"
}
myfunc "John"
```

## If else

```shell
if [[ -z ${APP_ENV} ]]; then
  echo "The APP_ENV should be set to prod or staging, got: $0"
  exit -1
fi

if [[ -z "$string" ]]; then
  echo "String is empty"
elif [[ -n "$string" ]]; then
  echo "String is not empty"
fi
```

## Condition

```shell
[[ -z STRING ]]						# Empty string
[[ -n STRING ]]						# Not empty string
[[ STRING == STRING ]]		# Equal
[[ STRING != STRING ]]		# Not Equal

[[ NUM -eq NUM ]]					# Equal
[[ NUM -ne NUM ]]					# Not equal
[[ NUM -lt NUM ]]					# Less than
[[ NUM -le NUM ]]	 				# Less than or equal
[[ NUM -gt NUM ]]					# Greater than
[[ NUM -ge NUM ]]					# Greater than or equal

[[ ! EXPR ]]							# Not
[[ X ]] && [[ Y ]]				# And
[[ X ]] || [[ Y ]]				# Or

[[ -e FILE ]]							# Exists
[[ -r FILE ]]							# Readable
[[ -h FILE ]]							# Symlink
[[ -d FILE ]]							# Directory
[[ -w FILE ]]							# Writable
[[ -s FILE ]]							# Size is > 0 bytes
[[ -f FILE ]]							# File
[[ -x FILE ]]							# Executable
```



## For loop

```shell
for i in *.tar; do echo working on $i; tar xvzf $i ; done

while true; do
  ···
done

for i in {1..5}; do
    echo "Welcome $i"
done
```

## Default value

```shell
${FOO:-val} 	# $FOO, or val if not set
${FOO:=val}		# Set $FOO to val if not set
```

## Auguments

```shell
$#	# Number of arguments
$*	# All arguments
$@	# All arguments, starting from first
$1	# First argument
```

## Reference

* [bash](https://devhints.io/bash)