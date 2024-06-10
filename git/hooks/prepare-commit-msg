#!/bin/sh
#
# An example hook script to prepare the commit log message.
# Called by "git commit" with the name of the file that has the
# commit message, followed by the description of the commit
# message's source.  The hook's purpose is to edit the commit
# message file.  If the hook fails with a non-zero status,
# the commit is aborted.
#
# To enable this hook, rename this file to "prepare-commit-msg".

min_length=5
max_length=50
BRANCH_PREFIX=(feature fix build docs refactor test bugfix chore hotfix)
BRANCHES_TO_SKIP=(master develop)
JIRA_PREFIX_REGEX_INPUT=${JIRA_PREFIX_REGEX:-""}-[0-9]{1,5}

function join { local IFS="$1"; shift; echo "$*"; }
function split { IFS='$1' read; echo "$*"; }

function build_regex() {
  skipExp=$(join '|' ${BRANCHES_TO_SKIP[@]})
  branchExp="($(join '|' ${BRANCH_PREFIX[@]}))\/${JIRA_PREFIX_REGEX_INPUT}[_-][A-Za-z0-9._-]{$min_length,$max_length}"
  branch_validation_expression="^($skipExp|($branchExp))$"
  commitexp="^([Rr]evert|[Mm]erge).+$"
}

# Print out a standard error message which explains
# how the commit message should be structured
function print_error() {
  branch_name=$1
  regular_expression=$2
  printf "\n\e[31m[Invalid Branch naming]\n"
  printf "\e------------------------\033[0m\e[0m\n"
  printf "Valid branch prefix: \e[36m%s\033[0m\n" "${BRANCH_PREFIX[*]}"
  printf "Branches skipped in check: \e[36m%s\033[0m\n" "${BRANCHES_TO_SKIP[*]}"
  printf "JIRA prefix : \e[36m$JIRA_PREFIX_REGEX_INPUT\033[0m\n"
  printf "Max length : \e[36m$max_length\033[0m\n"
  printf "Min length : \e[36m$min_length\033[0m\n"
  printf "\e[37mRegex: \e[33m$regular_expression\033[0m\n"
  printf "\e[37mActual branch name: \e[33m\"$branch_name\"\033[0m\n"
  printf "\e[37mActual length: \e[33m$(echo $branch_name | wc -c)\033[0m\n\n"
}

# get the first line of the commit message
INPUT_FILE=$1
START_LINE=`head -n1 $INPUT_FILE`
build_regex
BRANCH_NAME=$(git symbolic-ref --short HEAD)

if [[ ! $BRANCH_NAME =~ $branch_validation_expression ]]; then
  # commit message is invalid according to config - block commit
  print_error "$BRANCH_NAME" "$branch_validation_expression"
  exit 1
fi

BRANCH_TYPE=${BRANCH_NAME%/*}
BRANCH_EXCLUDED=$(printf "%s\n" "${BRANCHES_TO_SKIP[@]}" | grep -c "^$BRANCH_TYPE$")

# extract jira number
BRANCH_DETAILS=(${BRANCH_NAME//\// })
JIRA_TEMP=${BRANCH_DETAILS[1]}
[[ $JIRA_TEMP =~ ($JIRA_PREFIX_REGEX_INPUT) ]]
JIRA_NUMBER=${BASH_REMATCH[0]}
BRANCH_NAME_EXTRACT=${JIRA_TEMP#*$JIRA_NUMBER[_-]}


if ! [[ $BRANCH_EXCLUDED -eq 1 ]]; then
  if [[ $START_LINE =~ $commitexp ]]; then
    printf "Merge or revert commit"
    exit 0
  fi
  sed -i.bak -e "1s/^/[$JIRA_NUMBER][$BRANCH_TYPE][$BRANCH_NAME_EXTRACT] /" $1
  exit 0
fi

printf "no prefix added"
