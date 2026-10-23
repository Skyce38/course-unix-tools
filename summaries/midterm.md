# สรุปอ่านสอบ MIDTERM: Unix Tools and Programming

อิง [โครงร่างรายวิชา](https://angsila.cs.buu.ac.th/~nutthanon/88624159/) ที่ระบุสอบกลางภาคในสัปดาห์ 8 จึงจัดเนื้อหาสัปดาห์ 1–6 เป็นชุด Midterm การแบ่งนี้อิงลำดับเรียน ไม่ใช่ประกาศยืนยันขอบเขตข้อสอบของปีปัจจุบัน ตัวอย่างคำสั่งใช้ Linux/Bash

## 1. ภาพรวมที่ต้องอธิบายได้

- **Kernel** จัดการ CPU หน่วยความจำ อุปกรณ์ โพรเซส และระบบแฟ้ม
- **Shell** รับและตีความคำสั่ง เช่น Bash และใช้เขียน shell script
- **Utilities** โปรแกรมช่วยทำงาน เช่น `ls`, `cp`, `grep`
- **Multiuser** ผู้ใช้หลายคนใช้งานระบบร่วมกัน มี UID, GID และสิทธิของตน
- **Multitasking** ระบบจัดสรรเวลาประมวลผลให้หลายโพรเซส
- **Process** โปรแกรมที่กำลังทำงาน อ้างด้วย PID; **job** งานที่ shell ติดตาม อาจมีหลายโพรเซสใน pipeline เดียว

ลำดับการใช้งาน: ผู้ใช้ → shell → เรียกคำสั่ง/โปรแกรม → ขอใช้บริการ kernel

## 2. คำสั่งพื้นฐานและเส้นทาง

รูปแบบทั่วไป: `command [options] [arguments]` เช่น `ls -l /tmp`

| คำสั่ง | จำว่าใช้ทำอะไร |
|---|---|
| `man ls` | เปิดคู่มือคำสั่ง |
| `man 5 passwd` | เปิดคู่มือรูปแบบไฟล์ใน section 5 |
| `apropos password` | ค้นชื่อและคำอธิบายคำสั่งจาก keyword |
| `help cd` | คู่มือคำสั่งภายใน Bash |
| `pwd` | แสดง working directory |
| `ls -la` | แสดงรายละเอียด รวมชื่อที่ขึ้นต้นด้วยจุด |
| `cd dir` / `cd ..` | เข้า directory / ขึ้นไป directory แม่ |
| `cd` / `cd ~` | กลับ home directory |
| `cd -` | กลับ directory ก่อนหน้า |
| `cat a.txt` | แสดงเนื้อหาไฟล์ |
| `more a.txt` / `less a.txt` | อ่านทีละหน้า; `less` เลื่อนย้อนกลับได้ |
| `mkdir -p a/b` | สร้าง directory ซ้อนกัน |
| `rmdir empty` | ลบ directory ที่ว่าง |
| `cp a.txt b.txt` | คัดลอกไฟล์ |
| `cp -r src backup` | คัดลอก directory และเนื้อหา |
| `mv old new` | ย้ายหรือเปลี่ยนชื่อ |
| `rm a.txt` / `rm -r dir` | ลบไฟล์ / ลบ directory พร้อมเนื้อหา |
| `ssh user@host` | เข้าใช้เครื่องปลายทาง |
| `scp a.txt user@host:/tmp/` | คัดลอกไฟล์ไปเครื่องปลายทาง |

**Absolute path** เริ่มจาก `/` เช่น `/home/student/report.txt`

**Relative path** เริ่มจาก directory ปัจจุบัน เช่น `../report.txt`

`.` = ปัจจุบัน, `..` = directory แม่, `~` = home ของผู้ใช้; `/` คือ root directory ส่วน `/root` มักเป็น home ของผู้ใช้ root บน Linux

ตัวอย่าง: อยู่ที่ `/home/mali/work` แล้วสั่ง `cd ../notes` → อยู่ที่ `/home/mali/notes`

ใน `less`: Space = หน้าถัดไป, `b` = ย้อนหน้า, `/word` = ค้น, `n` = ผลถัดไป, `q` = ออก

## 3. vi: จำโหมดก่อนจำปุ่ม

| งาน | ปุ่ม/คำสั่ง |
|---|---|
| เปิดไฟล์ | `vi note.txt` |
| เข้า insert mode | `i` ก่อน cursor, `a` หลัง cursor, `o` เปิดบรรทัดใหม่ด้านล่าง |
| กลับ normal/command mode | `Esc` |
| เลื่อน | `h` ซ้าย, `j` ลง, `k` ขึ้น, `l` ขวา |
| ไปต้น/ท้ายบรรทัด | `0` / `$` |
| ไปท้ายไฟล์ | `G` |
| ไปบรรทัด 20 | `:20` |
| ลบ | `x` ลบตัวอักษร, `dd` ลบบรรทัด, `3dd` ลบ 3 บรรทัด |
| คัดลอกและวาง | `yy` คัดลอกบรรทัด, `p` วางหลังตำแหน่งปัจจุบัน |
| ย้อนการแก้ไข | `u` |
| ค้น | `/word` แล้ว Enter; `n` ไปผลถัดไป |
| บันทึก | `:w` |
| บันทึกแล้วออก | `:wq` |
| ออกเมื่อไม่มีการแก้ไขค้าง | `:q` |
| ออกทิ้งการแก้ไข | `:q!` |
| แทนคำทั่วไฟล์ | `:%s/old/new/g` |

แบบฝึกสั้น: เปิดไฟล์ → `i` → พิมพ์ → `Esc` → `:wq` → Enter

## 4. User, group และ permission

`id` แสดง UID, GID และกลุ่ม; `/etc/passwd` เก็บข้อมูลบัญชี, `/etc/group` เก็บข้อมูลกลุ่ม, `/etc/shadow` เก็บ password hash และข้อมูลอายุรหัสผ่าน

อ่าน `ls -l` ตัวอย่าง:

```text
-rwxr-x--- 2 mali students 120 Oct 17 10:00 run.sh
│└─┬──┘
│  permission: owner / group / others
└ file type: - ไฟล์ทั่วไป, d directory, l symbolic link
```

สิทธิแบ่งสามชุดตามลำดับ **u / g / o**; แต่ละชุดเรียง **r / w / x**

| สิทธิ | ไฟล์ทั่วไป | Directory |
|---|---|---|
| `r` | อ่านเนื้อหา | อ่านรายชื่อสมาชิก |
| `w` | แก้ไขเนื้อหา | เพิ่ม/ลบ/เปลี่ยนชื่อสมาชิก โดยทั่วไปต้องมี `x` ร่วมด้วย |
| `x` | เรียกเป็นโปรแกรม | เดินผ่าน/เข้าถึงชื่อภายใน directory |

**จุดสำคัญ:** การลบชื่อไฟล์พิจารณาสิทธิของ directory แม่เป็นหลัก จึงไม่ได้ตัดสินจาก `w` ของตัวไฟล์เพียงอย่างเดียว; sticky bit อาจเพิ่มข้อจำกัด

เลขสิทธิ: `r=4`, `w=2`, `x=1` แล้วบวกในแต่ละชุด

| เลข | สิทธิ | ตัวอย่าง |
|---|---|---|
| 7 | `rwx` | 4+2+1 |
| 6 | `rw-` | 4+2 |
| 5 | `r-x` | 4+1 |
| 4 | `r--` | 4 |
| 0 | `---` | ไม่มีสิทธิ |

```bash
chmod 754 run.sh       # rwxr-xr--
chmod u+x run.sh       # เพิ่ม x ให้เจ้าของ
chmod go-w data.txt    # ถอน w จาก group และ others
chown mali data.txt    # เปลี่ยนเจ้าของ ต้องมีสิทธิเพียงพอ
chgrp students data.txt
```

**umask:** สิทธิเริ่มต้น = ฐานสิทธิ AND NOT mask ไม่ใช่การลบเลขฐานแปดแบบธรรมดา

- ฐานทั่วไปของไฟล์ใหม่ `666`; directory ใหม่ `777`
- `umask 022` → ไฟล์ `644`, directory `755`
- `umask 027` → ไฟล์ `640`, directory `750`
- ไฟล์ปกติไม่ได้รับ `x` อัตโนมัติจากฐาน `666`

## 5. Process และ job control

```bash
ps aux           # รายการโพรเซส
sleep 60 &       # ทำงาน background
jobs             # งานที่ shell นี้ติดตาม
fg %1            # นำ job 1 กลับ foreground
bg %1            # ให้งานที่หยุดอยู่ทำต่อใน background
kill 1234        # ส่ง SIGTERM ให้ PID 1234
kill -9 1234     # ส่ง SIGKILL
```

- `Ctrl+C` ส่ง SIGINT ไปยังกลุ่มโพรเซส foreground ตามการตั้งค่า terminal ปกติ
- `Ctrl+Z` หยุดงานชั่วคราว แล้วใช้ `bg` หรือ `fg` ให้ทำต่อ
- `&` เริ่มงานแบบ background; ไม่ใช่ pipe
- `kill` ส่งสัญญาณ; ไม่ได้แปลว่าทุกสัญญาณต้องฆ่าโพรเซส
- **PID 1234** กับ **job `%1`** เป็นคนละระบบเลขอ้างอิง

## 6. Docker, บัญชีผู้ใช้ และเครือข่าย

อยู่ในสัปดาห์ 3 ของ outline จึงควรทบทวน แม้รายละเอียดที่ใช้จริงขึ้นกับห้องปฏิบัติการ

**Image** = แม่แบบสำหรับสร้าง container; **container** = instance ที่สร้างจาก image; **registry** = ที่จัดเก็บและแจกจ่าย image

```bash
docker images
docker ps                  # container ที่กำลังทำงาน
docker ps -a               # รวมที่หยุดแล้ว
docker run -it ubuntu bash
docker exec -it mybox bash  # เข้า container ที่กำลังทำงาน
docker stop mybox
docker start mybox
docker run -d -p 8080:80 nginx
```

`-i` เปิด stdin, `-t` จัดสรร pseudo-terminal, `-d` ทำงาน detached, `-p 8080:80` จับคู่ **port เครื่อง host : port container**

คำสั่ง Linux ที่ควรรู้จัก: `sudo command`, `passwd`, `sudo useradd -m student` และ `sudo passwd student` ส่วน `adduser` และคำสั่งตั้งค่าเครือข่ายต่างกันตาม distribution

เครือข่าย: IP = ที่อยู่เครื่อง, subnet/prefix = ขอบเขตเครือข่าย, gateway = ทางออกไปเครือข่ายอื่น, DNS = แปลงชื่อเป็น IP; `ip addr` ดูที่อยู่, `ip route` ดูเส้นทาง, `ping host` ทดสอบ ICMP reachability

## 7. อักขระพิเศษ: shell เป็นคนตีความ

| รูปแบบ | ความหมาย/ตัวอย่าง |
|---|---|
| `*` | glob: ตัวอักษรตั้งแต่ 0 ตัวขึ้นไป เช่น `*.txt` |
| `?` | glob: ตัวอักษรหนึ่งตัว เช่น `a?.txt` |
| `[abc]`, `[0-9]` | glob: หนึ่งตัวจากชุด/ช่วง |
| `~` | home เช่น `~/work` |
| `;` | จบคำสั่งหนึ่งแล้วทำคำสั่งต่อไป |
| `&` | ให้ทำงาน background |
| `\` | escape ตัวถัดไป; ก่อน newline ใช้ต่อบรรทัด |
| `'...'` | เก็บข้อความตรงตัว ไม่ขยาย `$var` |
| `"..."` | ยังขยายตัวแปรและ `$(...)` แต่ป้องกัน word splitting และ globbing ของผลขยาย |
| `$name`, `${name}` | แทนด้วยค่าตัวแปร |
| `$(command)` | นำ stdout ของคำสั่งมาแทนตำแหน่งนี้ |
| `$((expression))` | คำนวณจำนวนเต็ม เช่น `$((3+4))` |
| `{a,b}`, `{1..3}` | brace expansion สร้างหลายคำ |
| `( commands )` | ทำคำสั่งใน subshell |
| `{ commands; }` | จัดกลุ่มใน shell ปัจจุบัน; เว้นวรรคและมีตัวคั่นก่อน `}` |
| `!` | กลับ exit status ใน `! command`; ใน interactive shell อาจเป็น history expansion |

เครื่องหมาย backtick ใช้ command substitution ได้เช่นกัน แต่ `$(...)` อ่านและซ้อนคำสั่งง่ายกว่า ส่วน `^` มีความหมายตามบริบท เช่นต้นบรรทัดใน regex ไม่ใช่ glob ที่แปลว่าเริ่มต้นชื่อไฟล์

```bash
name=Dao
printf '%s\n' "$name"   # Dao
printf '%s\n' '$name'   # $name
printf '%s\n' \$name    # $name
printf '%s\n' file{1,2}.txt
# file1.txt
# file2.txt
```

glob มักไม่จับชื่อที่ขึ้นต้นด้วย `.` และถ้าไม่พบชื่อที่ตรง pattern ค่าเริ่มต้นของ Bash จะคง pattern นั้นไว้

## 8. ระบบแฟ้ม: directory → inode → data

```text
directory entry: ชื่อไฟล์ + หมายเลข inode
                         ↓
inode: ชนิด สิทธิ UID/GID ขนาด เวลา link count และตัวชี้ข้อมูล
                         ↓
data blocks: เนื้อหาของไฟล์
```

**ชื่อไฟล์อยู่ใน directory entry ไม่ได้เก็บเป็นชื่อหลักใน inode**; `ls -i` แสดงหมายเลข inode

| ประเด็น | Hard link | Symbolic link |
|---|---|---|
| สร้าง | `ln original alias` | `ln -s original shortcut` |
| หลักการ | เพิ่มชื่อที่อ้าง inode เดิม | ไฟล์ link เก็บ pathname ปลายทาง |
| inode ของ link | เดียวกับต้นฉบับใน filesystem เดียวกัน | มี inode ของตนเอง |
| ลบชื่อเดิม | อีกชื่อยังใช้ข้อมูลได้ | link อาจขาด ถ้า pathname เดิมใช้ไม่ได้ |
| ข้าม filesystem | ไม่ได้ | ได้ |

`rm` ถอน directory entry; พื้นที่ข้อมูลถูกคืนเมื่อไม่มี hard link และไม่มีโพรเซสเปิดไฟล์นั้นค้างอยู่ การเปลี่ยนชื่อภายใน filesystem เดียวกันโดยทั่วไปไม่ต้องคัดลอกข้อมูลใหม่

### สูตร pointer ตามแบบจำลองในสไลด์

สมมติ inode มี **12 direct + single indirect + double indirect + triple indirect**; ให้ block size = `B` bytes, pointer size = `P` bytes และ `N=B/P`

```text
direct           = 12B
single indirect  = NB
double indirect  = N²B
triple indirect  = N³B
รวมความจุข้อมูล = B(12 + N + N² + N³)
```

เช่น `B=4096`, `P=4` → `N=1024` → รวม `48 KiB + 4 MiB + 4 GiB + 4 TiB` โดยไม่นับพื้นที่บล็อกเก็บ pointer ในยอดข้อมูล แบบจำลองนี้ใช้ทำโจทย์ของสไลด์ ไม่ใช่สูตรของทุก filesystem

โจทย์พื้นที่: ไฟล์ 13 KiB บน block 4 KiB ต้องใช้ data block `ceil(13/4)=4` บล็อก รวม 16 KiB เหลือในบล็อกท้าย 3 KiB โดยไม่นับ metadata และสมมติไฟล์ไม่เป็น sparse file

## 9. Redirection และ pipeline

File descriptor พื้นฐาน: **0 = stdin, 1 = stdout, 2 = stderr**

| รูปแบบ | ผล |
|---|---|
| `cmd < in.txt` | รับ stdin จากไฟล์ |
| `cmd > out.txt` | ส่ง stdout ไปไฟล์ สร้างใหม่/ตัดเนื้อหาเดิม |
| `cmd >> out.txt` | ต่อท้าย stdout |
| `cmd 2> err.txt` | ส่ง stderr ไปไฟล์ |
| `cmd > all.txt 2>&1` | ส่ง stdout ไปไฟล์ แล้วให้ stderr ไปที่เดียวกับ stdout |
| `cmd 2>/dev/null` | ทิ้ง stderr |
| `a \| b` | stdout ของ a เป็น stdin ของ b |
| `a \| tee out.txt` | ส่งข้อมูลไปทั้ง stdout และไฟล์ |

**ลำดับ redirection อ่านจากซ้ายไปขวา**

```bash
cmd > all.txt 2>&1   # ทั้ง stdout และ stderr ไปไฟล์
cmd 2>&1 > out.txt   # stderr ไปปลายทาง stdout เดิม; stdout ไปไฟล์
```

`a | b` ต่อเฉพาะ stdout ตามปกติ; pipeline ทั้งชุดสามารถทำ background ด้วย `a | b &`

### Utilities ที่ใช้ต่อกันบ่อย

| คำสั่ง | หน้าที่ |
|---|---|
| `grep 'word' file` | เลือกบรรทัดที่ตรง pattern |
| `grep -v 'word' file` | เลือกบรรทัดที่ไม่ตรง |
| `grep -n 'word' file` | แสดงเลขบรรทัดด้วย |
| `sort file` / `sort -n file` | เรียงข้อความ / เรียงตามตัวเลข |
| `sort -r file` | เรียงย้อนลำดับ |
| `uniq` / `uniq -c` | ยุบ/นับบรรทัดซ้ำที่อยู่ติดกัน |
| `wc -l` / `wc -w` / `wc -c` | นับ newline / คำ / bytes |
| `head -n 5` / `tail -n 5` | ห้าบรรทัดแรก / ท้าย |
| `cut -d: -f1` | เลือก field แรก ใช้ `:` คั่น |
| `tr '[:lower:]' '[:upper:]'` | เปลี่ยนตัวเล็กเป็นใหญ่ |
| `xargs command` | แปลง stdin เป็น arguments ของคำสั่ง |

```bash
grep 'ERROR' app.log | wc -l
# จำนวนบรรทัดที่มี ERROR ไม่ใช่จำนวนครั้งที่ ERROR ปรากฏ

sort names.txt | uniq -c | sort -nr
# รวมชื่อเหมือนกัน → นับ → เรียงจำนวนมากไปน้อย

cut -d: -f1 /etc/passwd | sort
# ชื่อบัญชีเรียงตามลำดับ
```

**Pipe ส่งเป็นข้อมูล stdin ส่วน `$(...)` ส่งผลลัพธ์มาเป็นส่วนของบรรทัดคำสั่ง** เช่น `wc -l < a.txt` อ่านเนื้อหา a.txt; `echo "$(wc -l < a.txt)"` นำจำนวนมาพิมพ์

## 10. แบบฝึกหัดแต่งเพิ่มพร้อมเฉลย

ลองตอบก่อนอ่านเฉลย ชุดนี้ใช้ฝึกตามหัวข้อในเว็บ ไม่ใช่ข้อสอบเก่าที่ได้รับการยืนยัน

1. แปลง `-rw-r-----` เป็นเลข permission → **640**
2. `chmod 751 x` ได้สิทธิอะไร → **rwxr-x--x**
3. ตั้ง `umask 027` แล้วสร้าง directory ปกติ ได้อะไร → **750**
4. ต้องการ stdout และ stderr อยู่ใน `run.log` → **`cmd > run.log 2>&1`**
5. `a; b` กับ `a | b` ต่างกันอย่างไร → **แบบแรกทำตามลำดับ; แบบหลังต่อ stdout ของ a เข้า stdin ของ b**
6. ข้อมูล `b, a, b` คนละบรรทัด ใช้ `uniq` อย่างเดียวจะนับ b รวมได้ไหม → **ไม่ได้ ต้องจัดบรรทัดเหมือนกันให้อยู่ติดกัน เช่น `sort | uniq -c`**
7. สร้าง hard link แล้วลบชื่อเดิม ข้อมูลหายทันทีไหม → **ไม่หายถ้ายังมี hard link อื่น**
8. inode เก็บชื่อไฟล์หรือไม่ → **ไม่เก็บชื่อหลัก; directory entry เชื่อมชื่อกับ inode**
9. กด `Ctrl+Z` แล้วอยากทำงานนั้นต่อเบื้องหลัง → **`bg %หมายเลขงาน`**
10. มี `a1.txt`, `a2.txt`, `a10.txt` แล้ว `a?.txt` ตรงกับอะไร → **a1.txt และ a2.txt**
11. `echo '*.txt'` ขยายเป็นรายชื่อไฟล์ไหม → **ไม่ขยาย เพราะ single quote**
12. block 2 KiB, pointer 4 bytes: single indirect เก็บข้อมูลได้เท่าไร → **512 × 2 KiB = 1 MiB**

## อ่านรอบสุดท้าย

- [ ] แปลง rwx ↔ เลขฐานแปด และแยกสิทธิไฟล์กับ directory ได้
- [ ] อธิบายชื่อไฟล์, inode, data block และ hard/symbolic link ได้
- [ ] คำนวณ direct/indirect จากขนาด block และ pointer ได้
- [ ] อ่าน quote/glob และแยก `;`, `&`, `|`, `$(...)` ได้
- [ ] ใช้ stdin/stdout/stderr และอธิบายลำดับ `2>&1` ได้
- [ ] ต่อ pipeline นับ กรอง เรียงข้อมูลได้
- [ ] ใช้ vi บันทึก/ออก และ job control ได้
- [ ] ทบทวน Docker และคำสั่งจัดการบัญชีตามที่ทำใน lab

## แหล่งประกอบ

- [หน้าเนื้อหารายวิชาและจุดแบ่งสอบ](https://angsila.cs.buu.ac.th/~nutthanon/88624159/)
- [Lecture 1: ระบบปฏิบัติการ ผู้ใช้และสิทธิ](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect01.pdf)
- [Lecture 2](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect%2002.pdf)
- [Lecture 3 ตามป้ายในหน้าเว็บ: command line](https://www.dropbox.com/s/4saquh8dyt6fqs5/Lect04.pdf?dl=0)
- [Lecture 4 ตามป้ายในหน้าเว็บ: อักขระพิเศษ](https://www.dropbox.com/s/n9fudbxyupie7me/Lect04_2.docx?dl=0)
- [เอกสารระบบแฟ้มที่ลิงก์ในสัปดาห์ 5](https://www.dropbox.com/s/1szrdcr08pz1e93/Lect06.pdf?dl=0)
- [เอกสาร Redirection/Pipeline ในสัปดาห์ 6](https://www.dropbox.com/s/txwafybaeuoukhh/Lect06.pdf?dl=0)
- [Docker ที่รายวิชาลิงก์ไว้](https://docs.google.com/presentation/d/10Ab2i7QeSAQgMwgncuLkiB1gV5J3MRA08Qqa9Q_qm3c/htmlpresent)

หมายเหตุ: ป้ายชื่อ lecture บนเว็บกับชื่อไฟล์บางรายการไม่ตรงกัน สรุปนี้จัดตามเนื้อหาที่อ่านได้จริง ตัวอย่างและคำอธิบายเรียบเรียงใหม่เพื่อทบทวน
