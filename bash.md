# Bash Snippets

## Table Of Contents
  * [get script parameter](#get-script-parameter)
  * [user input](#user-input)
  * [REGEX](#REGEX)
  * [number checks](#number-checks)
  * [Array](#Array)
  * [array item count](#array-item-count)
  * [loop though an array](#loop-though-an-array)
  * [extract text](#extract-text)
  * [network cards with MAC address](#network-cards-with-MAC-address)
  * [loop through all template and user folders](#loop-through-all-template-and-user-folders)


## get script parameter
<pre>
# get parameter
while getopts ":i:o:n:h:" o; do
    case "${o}" in
        i)
			input="${OPTARG}"
            ;;
        o)
			output="${OPTARG}"
            ;;
        n)
			name="${OPTARG}"
            ;;
        h)
		    echo "help"
		    showhelp=1
            ;;
        *)
		    showhelp=1
            ;;
    esac
done
shift $((OPTIND-1))

# help
if [ ! -z "${showhelp}" ]; then
	echo ""
	echo "Syntax: ${0} -n NAME -i INPUTFILE [-o OUTPUTFILE]"
	echo "	-n		Name foo
	echo "	-i		input file"
	echo ""
	echo "Optional:"
	echo "	-o		output file"
	exit 1
fi
</pre>

## user input
<pre>
if [ -z "${apiuser}" ] || [ -z "${apipass}" ]; then
	echo "Please enter your credentials"
	printf "Username: "
	read apiuser
	printf "Passwort: "
	stty -echo # disable screen output
	read apipass
	stty echo # enable screen output
	echo ""
fi

if [ -z "${apiuser}" ] || [ -z "${apipass}" ]; then
	echo "credentials not entered"
	exit 1
fi
</pre>

## REGEX
<pre>
# possible computer names
# ABC12345
# abc98765
# xyz34567a
# ZYX87654H
REGEX="^(ABC|abc)[[:digit:]]{8}[a-zA-Z]{0,1}$"
if [[ ! ${Computer_Name} =~ ${REGEX} ]]; then
	echo "\"${Computer_Name}\" is correct."
fi
</pre>

## number checks
<pre>
i=123
number="^[[:digit:]]{1,}$"
if [[ ${i} =~ ${number} ]]; then
	echo "input »${i}« is a number"
else
	echo "input »${i}« is not a number"
fi
</pre>

<pre>
i="123"
i=${i/.*}
echo "${i}"
if [[ $i =~ ^-?[0-9]+$ ]]; then
	echo "»${i}« is integer"
else
	echo "»${i}« is not integer"
fi
</pre>

## Array
<pre>
array=("foo")
array+=("bar")
array+=("baz")

# array item count
echo "${#array[@]}" # output: 3

# loop though an array
for i in ${array[@]}
do
	echo "${i}"
done
# output:
# foo
# bar
# baz

# array ids
echo "${!array[@]}"
</pre>

## extract text
<pre>
echo "foo bar baz" | sed "s/.* \(.*\) .*/\1/g" # output: bar
</pre>

## network cards with MAC address
<pre>
ports=$(networksetup -listallhardwareports | grep "Hardware Port:" | sed "s/Hardware Port: //")
IFS=$'\n' read -rd '' -a port <<<"${ports}"

portsMAC=$(networksetup -listallhardwareports | grep "Ethernet Address:" | sed "s/Ethernet Address: //")
IFS=$'\n' read -rd '' -a portMAC <<<"${portsMAC}"

echo "Netword cards with MAC address"
echo "=============================="
i=0
for pMAC in "${portMAC[@]}"
do
	if [[ "${pMAC}" != *"N/A"* ]]; then
		echo "${port[$i]}"
	fi
	i=$(( $i+1 ))
done

exit 0
</pre>

## loop through all template and user folders
<pre>
FOLDER_ARRAY=("/System/Library/User Template/English.lproj")
FOLDER_ARRAY+=("/System/Library/User Template/German.lproj")

for USER_HOME in /Users/*
do
	USER_UID=$(basename "${USER_HOME}")
	if [ ! "${USER_UID}" = "Shared" ]; then
		FOLDER_ARRAY+=("${USER_HOME}")
	fi
done

for ((i = 0; i < ${#FOLDER_ARRAY[*]}; i++))
do
	USER_HOME=${FOLDER_ARRAY[$i]}
	USER_UID=`basename "${USER_HOME}"`
	if [ ! "${USER_UID}" = "Shared" ] && [ ! "${USER_UID}" = "Guest" ]; then
		touch "${USER_HOME}/Desktop/foo"
		if [ ! "${USER_UID}" = "English.lproj" ] && [ ! "${USER_UID}" = "German.lproj" ]; then
			chown -R "${USER_UID}" "${USER_HOME}/Desktop/foo"
		fi
		echo ""
	fi
done
</pre>