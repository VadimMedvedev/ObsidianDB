
`channel-group <номер канала> mode desirable|active` - добавляет порт в канал (до 8 портов в канале) и автоматически на другой стороне. Теперь везде заместо них будет показываться объединённый port-channel (Po). Mode указывает тип протокола:
* desirable - PAgP
* active - LACP
`interface channel-group <номер>` - переход в его настройки. По сути его можно настраивать ровно также как и обычные порты.
`show etherchannel summary` - показывает инфу про установленные каналы

int ra f0/21-22
channel-group 6 mode active
int po 6
sw m trunk
sw tr na vlan 99
sw tr al vlan 10,20

sp m r
sp v 10 p 32768
sp v 20 p 32768