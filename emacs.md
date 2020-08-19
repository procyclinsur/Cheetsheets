### golang mono-repo go.mod projects and autocomplete (spacemacs)

To get `helm` (autocomplete engine) working properly with `go-mode` and `go.mod` projects in a mono-repository environment one must complete the following.

- create a workspace folder for the directory you will be working in `spc-m-F-a` (space-mode(go-mode)-folder-add) (lsp-workspace-folder-add)
- switch to the created workspace `spc-m-F-s` (space-mode(go-mode)-folder-switch) (lsp-workspace-folder-switch) **unsure if this is completely necessary**
- restart lsp server `spc-m-b-r` (space-mode(go-mode)-backend-restart)
