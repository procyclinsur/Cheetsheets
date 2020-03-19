# Dev.to related tricks

## PlantUML Theme

```plantuml
@startuml

!define Background   #141F2D
!define Participant  #22303F
!define Border       #424A54
!define TextColor    #DDDDDD

skinparam Shadowing false
skinparam roundcorner 20
skinparam backgroundColor Background
skinparam Arrow {
  Color TextColor
  FontColor TextColor
  FontStyle Bold
}
skinparam Default {
  FontName Courier
  FontColor TextColor
}
skinparam sequence {
  LifeLineBorderColor TextColor
  GroupBorderColor TextColor
  GroupBorderThickness 1
  GroupBackgroundColor Participant
  GroupFontColor TextColor
  GroupHeaderFontStyle Normal
  GroupFontStyle Normal
}
skinparam participant {
  BackgroundColor Participant
  BorderColor Border
  FontColor TextColor
}

title Query Job Completion

participant client as c
participant server as s

c -> s : 01E3S4Q9SN3VHVXB2KCAGD9P62
alt job finished
    s -> c : https://link.to.your/output.csv
else running/error
    s -> c : Error: job still running
end
@enduml
```
