
### Модуль community.general.parted

Модуль позволяет создавать таблицу разделов на дисках при помощи утелиты `parted`.
Для того чтобы этот модуль работал, на управляемой машине `parted` должен быть установлен.

##### Параметры
* `device` - **обязательный**, в нём указывается путь до файла устройства (`/dev/vdb`)
* `label` - выбор типа [[Таблицы разделов|таблицы разделов]]. Если тип таблицы отличается от существующего, то он изменится, при этом удалятся все разделы на нём. Основные два возможные типа:
	1. `gpt` - таблица разделов GPT
	2. `msdos` - таблица разделов MBR
* `number` - указание номера раздела
* `state` - может принимать три состояния (очевидно для этого надо указать номер раздела `number`):
	1. `info` - выводит информацию о разделе
	2. `absent` - удалить этот раздел
	3. `present` - создать этот раздел
* `part_start` и `part_end` - указывают начало и конец раздела (используются для создания).

##### Примеры
``` yaml
- name: Create partition table on /dev/vdb  
  community.general.parted:  
    device: /dev/vdb  
    label: gpt  
    state: present  
  
- name: Create 25GB partition  
  community.general.parted:  
	device: /dev/vdb  
	number: 1  
	state: present  
	part_start: 1MiB  
	part_end: 25GiB
	
- name: Remove partition number 1
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: absent
```

### Модуль community.general.lvg

Этот модуль позволяет создавать [[Система управления томами LVM|VG]]. При этом создание PV происходит в автоматическом режиме, если они ещё не созданы.

##### Параметры
* `vg` - название volume proup
* `state` - основные два состояния:
	1. `present` - создаёт, если не существует (по умолчанию)
	2. `absent` - удаляет, если существует
* `pvs` - список через запятую PV которые войдут в группу при её создании. Если VG уже существует, то эти PV просто добавятся к группе. Если на каком-то устройстве ещё не создан PV, то он автоматически создастся
* `remove_extra_pvs` - булевое значение (yes/no). Если активировано, то все не указанные в параметре `pvs` физические тома будут удалены (по умолчанию true).

##### Примеры
``` yaml
- name: Create or resize a volume group on top of /dev/sdb1 and /dev/sdc5.
  community.general.lvg:
    vg: vg.services
    pvs:
      - /dev/sdb1
      - /dev/sdc5
        
- name: Remove a volume group with name vg.services
  community.general.lvg:
    vg: vg.services
    state: absent
    
- name: Add new PVs to volume group without removing existing ones
  community.general.lvg:
    vg: vg.services
    pvs: /dev/sdb1,/dev/sdc1
    remove_extra_pvs: false
    state: present
```

##### Замечания
Чтобы добавить тома и не удалить старые из VG, необходимо указать `remove_extra_pvs: no`.
Создавать PV через специальный модуль lvm_pv не рекомендуется, т.к. это уже попахивает императивом (ansible использует декларативный язык) а также приводит к ошибкам.

### Модуль community.general.lvol

Используется для создания и модификации LV в LVM.

##### Параметры
* `vg` - **обязательно**, с LV в какой VG мы работаем.
* `lv` - название LV
* `state` - может принимать два состояния:
	1. `present` - для создания (по умолчанию)
	2. `absent` - для удаления LV
* `snapshot` - используется для создания [[Снепшоты в LVM|снепшота]], указыввается название снепшота, в `lv` оригинал
* `resizefs` - при изменеии размера LV можно включить автоматическое расширение ФС на нём (по умолчанию false).
* `size` - указание размера создаваемоего диска, или изменение размера (если существует), указывается также как и в [[Работа с LVM|lvcreate]]. Если указать `+` то размер будет увеличин на заданное число, или же если `-` то уменьшен.

##### Примеры
``` yaml
- name: Create a logical volume the size of all remaining space in the volume group
  community.general.lvol:
    vg: firefly
    lv: test
    size: 100%FREE

- name: Extend the logical volume by given space
  community.general.lvol:
    vg: firefly
    lv: test
    size: +512M
```

### Модуль community.general.filesystem

Позволяет создавать ФС на устройствах.

##### Параметры
* `dev` - путь до файла устройства (`/dev/vdb1`)
* `fstype` - указание требуемой ФС (возможные значения `ext2`, `ext3`, `ext4`, `xfs`, `vfat` и т.д.).
* `force` - при true будет позволять менять ФС, если уже есть другая ФС (по умолчанию false).

### Модуль ansible.posix.mount

Монтирование файловых систем. При этом изменения применяются в файл `/etc/fstab`, т.е. сохраняется после перезагрузки.

##### Параметры 
* `path` - место монтирования (**необходимо**)
* `src` - монтируемое устройство (`/dev/vdb1` или `UUID=b3e48f45-...`)
* `state` - состояние (что сделать?):
	* `mounted` - примонтировать, сохраняется при перезагрузке (т.е. также настраивает `fstab`)
	* `unmounted` - отмонтировать (не меняет `fstab`), не обязательно указывать `src`
	* `ephemeral` - разовое монтирование, без конфигурации `fstab`.
	* `present` - просто проверка есть ли конфигурация устройства в `fstab`
	* `absent` - полное удаление монтирования и в этой сесси и в `fstab` (можно не указывать `src`)
* `opts` - указание опций (через запятую без пробелов) как при [[Монтирование блочных устройств|обычном монтировании]].
* `fstype` - указание типа ФС, обязательно при монтировании (`mounted`, `ephemeral`, `present`)

##### Примеры
``` yaml
- name: Mount up device by UUID
  ansible.posix.mount:
    path: /home
    src: UUID=b3e48f45-f933-4c8e-a700-22a159ec9077
    fstype: xfs
    opts: noatime
    state: present
    
- name: Unmount a mounted volume
  ansible.posix.mount:
    path: /tmp/mnt-pnt
    state: unmounted
```