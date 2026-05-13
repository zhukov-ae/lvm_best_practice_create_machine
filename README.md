# lvm_best_practice_create_machine
Для сервера Ubuntu с диском 25 ГБ оптимальная схема зависит от роли сервера, но для большинства случаев best practice — использовать:

* отдельный `/boot`;
* LVM для остальных разделов;
* не выделять сразу всё пространство под корень;
* оставить свободное место в VG для будущего расширения и snapshot.

---

# Рекомендуемая схема для Ubuntu Server 25 GB

```text id="2rz3dw"
Диск /dev/sda — 25 GB

┌──────────────────────────────────────┐
│ /boot        1 GB    ext4            │
├──────────────────────────────────────┤
│ LVM PV       ~24 GB                  │
│                                      │
│   VG ubuntu-vg                       │
│                                      │
│   ├── lv_root   12 GB   ext4/xfs     │
│   ├── lv_var     5 GB   ext4/xfs     │
│   ├── lv_log     3 GB   ext4/xfs     │
│   ├── lv_swap    2 GB   swap         │
│   └── FREE       ~2 GB               │
└──────────────────────────────────────┘
```

---

# Почему именно так

## `/boot` отдельно

GRUB и initramfs работают стабильнее при отдельном обычном разделе.

Обычно:

* 1 GB более чем достаточно;
* ext4;
* без LVM.

---

# Использование LVM

LVM позволяет:

* расширять разделы без переразметки;
* делать snapshots;
* переносить данные;
* добавлять диски позже.

---

# Разделение `/`, `/var`, `/log`

## `/` (root)

Система и приложения.

12 GB обычно хватает Ubuntu Server + Docker + пакеты.

---

## `/var`

Здесь растут:

* apt cache;
* docker overlay;
* базы;
* spool;
* systemd;
* journald.

Выделять отдельно — best practice для серверов.

---

## `/var/log`

Очень полезно отдельно.

Если логи внезапно забьют диск:

* система не упадёт полностью;
* root останется жив;
* проще чистить.

---

## swap

Для 25 GB диска:

* 2 GB достаточно;
* если будет hibernate — swap >= RAM.

---

# Почему оставлять FREE space в VG

Очень важный best practice.

```text id="kk1vfw"
НЕ выделять сразу 100% пространства.
```

Оставьте 10–20%.

Это позволяет:

* быстро увеличить `/var`;
* сделать snapshot;
* временно расширить root;
* пережить рост логов.

---

# Для Docker/Containers

Если сервер под Docker/K8s:

Лучше:

```text id="utn3h9"
/var        8-10 GB
```

потому что Docker хранит данные в:

```text id="k7n9sx"
/var/lib/docker
```

---

# Для маленького сервера можно проще

Минимально хорошая схема:

```text id="m44mko"
/boot     1 GB
/root     15 GB
swap       2 GB
FREE       7 GB
```

---

# Файловая система

## ext4

Лучший универсальный вариант.

Плюсы:

* надёжность;
* простота;
* easy recovery;
* resize.

---

## xfs

Хорош для:

* логов;
* больших файлов;
* high IO.

Но:

* нельзя уменьшать.

Для Ubuntu small server чаще выбирают ext4.

---

# Как сделать при установке Ubuntu

В installer:

```text id="ubg8w2"
Storage configuration
→ Custom storage layout
```

Далее:

1. Создать GPT
2. Создать `/boot`
3. Остальное → Physical volume for LVM
4. Создать VG
5. Создать LV

---

# Пример именования

```text id="1k6t5w"
VG: ubuntu-vg

LV:
lv_root
lv_var
lv_log
lv_swap
```

---

# Итоговая best practice схема

```text id="4vk08n"
25 GB Disk
│
├─ /boot          1 GB
│
└─ LVM PV        24 GB
     │
     └─ VG ubuntu-vg
          ├─ lv_root   12 GB
          ├─ lv_var     5 GB
          ├─ lv_log     3 GB
          ├─ lv_swap    2 GB
          └─ FREE       2 GB
```

Это наиболее безопасная и гибкая схема для Ubuntu Server с LVM.
