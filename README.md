The Command Line Murders
========================

	.OOOOOOOOOOOOOOO @@                                   @@ OOOOOOOOOOOOOOOO.
	OOOOOOOOOOOOOOOO @@                                    @@ OOOOOOOOOOOOOOOO
	OOOOOOOOOO'''''' @@                                    @@ ```````OOOOOOOOO
	OOOOO'' aaa@@@@@@@@@@@@@@@@@@@@"""                   """""""""@@aaaa `OOOO
	OOOOO,""""@@@@@@@@@@@@@@""""                                     a@"" OOOA
	OOOOOOOOOoooooo,                                            |OOoooooOOOOOS
	OOOOOOOOOOOOOOOOo,                                          |OOOOOOOOOOOOC
	OOOOOOOOOOOOOOOOOO                                         ,|OOOOOOOOOOOOI
	OOOOOOOOOOOOOOOOOO @          THE                          |OOOOOOOOOOOOOI
	OOOOOOOOOOOOOOOOO'@           COMMAND                      OOOOOOOOOOOOOOb
	OOOOOOOOOOOOOOO'a'            LINE                         |OOOOOOOOOOOOOy
	OOOOOOOOOOOOOO''              MURDERS                      aa`OOOOOOOOOOOP
	OOOOOOOOOOOOOOb,..                                          `@aa``OOOOOOOh
	OOOOOOOOOOOOOOOOOOo                                           `@@@aa OOOOo
	OOOOOOOOOOOOOOOOOOO|                                             @@@ OOOOe
	OOOOOOOOOOOOOOOOOOO@                               aaaaaaa       @@',OOOOn
	OOOOOOOOOOOOOOOOOOO@                        aaa@@@@@@@@""        @@ OOOOOi
	OOOOOOOOOO~~ aaaaaa"a                 aaa@@@@@@@@@@""            @@ OOOOOx
	OOOOOO aaaa@"""""""" ""            @@@@@@@@@@@@""               @@@|`OOOO'
	OOOOOOOo`@@a                  aa@@  @@@@@@@""         a@        @@@@ OOOO9
	OOOOOOO'  `@@a               @@a@@   @@""           a@@   a     |@@@ OOOO3
	`OOOO'       `@    aa@@       aaa"""          @a        a@     a@@@',OOOO'


There's been a murder in Terminal City, and TCPD needs your help.

To figure out whodunit, you need access to a command line.

Once you're ready, clone this repo, or [download it as a zip file](https://github.com/veltman/clmystery/archive/master.zip).

Open a Terminal, go to the location of the files, and start by reading the file 'instructions'.

One way you can do this is with the command:

	cat instructions

(`cat` is a command that will print the contents of the file called `instructions` for you to read.)

To get started on how to use the command line, open cheatsheet.md or cheatsheet.pdf (from the command line, you can type 'nano cheatsheet.md').

Don't use a text editor to view any files except these instructions, the cheatsheet, and hints.

### Credits

By Noah Veltman  
Projects: [noahveltman.com](http://noahveltman.com)  
GitHub: [veltman](https://github.com/veltman)  
Twitter: [@veltman](https://twitter.com/veltman)  

#! /bin/bash

getClues() {
    CLUES=$(grep -i CLUE mystery/crimescene)
    printf "Found some clues:\n$CLUES\n\n"

    # sort membership files for use by checkMembership
    MEMBERSHIPS=("AAA" "Delta_SkyMiles" "Terminal_City_Library" "Museum_of_Bash_History")
    for m in ${MEMBERSHIPS[@]}; do
        # sort files
        filePath="mystery/memberships/$m"
        sort "$filePath" > "$filePath.sorted"
    done
    COMMON_NAMES=$(comm -12 mystery/memberships/AAA.sorted mystery/memberships/Delta_SkyMiles.sorted | comm -12 - mystery/memberships/Terminal_City_Library.sorted | comm -12 - mystery/memberships/Museum_of_Bash_History.sorted)
    printf "These people have each membership:\n$COMMON_NAMES\n\n"
    while read -r line; do
        searchAddress "$line"
    done <<< "$COMMON_NAMES"
}

checkAnnabel() {
    INFO=$(grep Annabel mystery/people | cut -f1 -d $'\t')
    while read -r line; do
        NAME=$(echo $line | cut -f1 -d $'\t')
        searchAddress "$NAME"
        echo ""
    done <<< "$INFO"

    # found out that the catr is a blue Honda, license place starts with L337 and ends with 9
    matchCarWithPerp
}

searchAddress() {
    # address, line #
    INFO=$(grep "$1" mystery/people | cut -f4 -d $'\t')
    ADDRESS=$(echo $INFO | cut -f1 -d $',')
    LINE=$(echo $INFO | cut -f2 -d $',' | cut -f3 -d' ')

    # replace space with underscore
    ADDRESS=${ADDRESS// /_}
    FILE="mystery/streets/$ADDRESS"
    if [[ -e $FILE ]]; then
        # get interview number
        interview=$(sed "${LINE}q;d" $FILE)
        INT_NUM=$(echo $interview | cut -f2 -d'#')
        INTERVIEW_FILE="mystery/interviews/interview-$INT_NUM"
        if [[ -e $INTERVIEW_FILE ]]; then
            printf "Interview for $1:\n"
            searchInterview "$INTERVIEW_FILE"
        fi
    fi
}

searchInterview() {
    contents=$(<"$1")
    echo $contents
}


matchCarWithPerp() {
    # license number, blue honda, at least 6'
    MATCHES=$(egrep -A 5 ".*L337.*9" mystery/vehicles | egrep -B 1 -A 4 "Honda" | egrep -B 2 -A 3 Blue | egrep -B 4 -A 1 "6'")
    # echo $MATCHES
    OWNERS=$(echo "$MATCHES" | grep Owner | cut -f2 -d $':')

    while read -r line; do
        checkMemberships "$line"
    done <<< "$OWNERS"

}

# specific clue investigations
checkMemberships() {
    # investigate anyone with every membership
    RESULT=$(echo $COMMON_NAMES | grep "$1" | wc -l)
    if [[ $RESULT -gt 0 ]]; then
        # this person is in all memberships, owns the correct car, and is the correct height
        echo "$1 is very suspicious!"
    fi
}

checkAnswer() {
    echo "Jeremy Bowers" | $(command -v md5 || command -v md5sum) | grep -qif /dev/stdin encoded && echo CORRECT\! GREAT WORK, GUMSHOE. || echo SORRY, TRY AGAIN.
}

showPerson() {
    echo "$(grep "$1" mystery/people)"
    echo "$(grep -B 3 -A 3 "$1" mystery/vehicles)"
}

getClues
printf "\n\nInvestigating Annabel...\n"
checkAnnabel

printf "Jeremy Bowers is male and meets all criteria. Checking...\n\n"
showPerson "Jeremy Bowers"
checkAnswer
