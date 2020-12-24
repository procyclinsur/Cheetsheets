# Contains random scripts in bash

Scripts are tagged with topics for things I don't want to forget

tags: #dnf #column #read #here_string

this is a horribly inefficient comparitor for checking package update version differneces
but serves as a good example for the `column` command.
```bash
# func: update-diff  - displays the version diff for updates to packages via dnf
function update-diff () {
    local CURRENT=$(dnf list installed -q | uniq -w30 | awk '{ print $1,$2 }')
    local NEW=$(dnf check-update -q | sed -n '/^Obsoleting/q;p' | awk '{ print $1,$2 }')

    local UPDATE
    while IFS= read -r n; do
        while IFS= read -r c; do
            if [[ ${c%" "*} == ${n%" "*} ]]; then
                local UPDATE="$UPDATE"${n%" "*}" "${c#*" "}" --> "${n#*" "}"\n"
            fi
        done <<<$CURRENT
    done <<<$NEW

    column --table \
           --table-columns PACKAGE,CURRENT_VERSION," ",UPDATE \
           <<<$(echo -e "$UPDATE")
}
```
