# สรุปอ่านสอบ FINAL: Unix Tools and Programming

**อัปเดตตามรูปแบบสอบที่ผู้เรียนแจ้ง: ข้อสอบปรนัย** ให้อ่านโดยฝึกเลือกผลลัพธ์ของโค้ด ตรวจจำนวน arguments, exit status และเงื่อนไขขอบเขต คู่กับ [วิเคราะห์การบ้าน Final ทั้ง 6 ไฟล์](../assignments/final-homework-analysis.md) และ [ข้อฝึกปรนัย 40 ข้อพร้อมเฉลยอธิบาย](../exams/practice-40.md)

อิง [โครงร่างรายวิชา](https://angsila.cs.buu.ac.th/~nutthanon/88624159/) ช่วงหลังกลางภาค สัปดาห์ 9–13: Shell Script → ตัวแปร/test/if → ลูป → case/function → array/sed/awk การจัดชุดนี้อิงลำดับเรียน ไม่ใช่ประกาศยืนยันว่า Final จะไม่ออกเนื้อหาก่อนกลางภาค

ใช้ **Bash บน Linux** เป็นหลัก โดยเฉพาะ `[[ ]]`, array และ `local`; ใช้ `bash script.sh` หรือ shebang ที่ระบุ Bash ไม่ใช้ `sh` แทนโดยสมมติว่าทำงานเหมือนกันทั้งหมด

## 1. เริ่มเขียนและรัน shell script

```bash
#!/usr/bin/env bash
printf 'Hello %s\n' "$1"
```

บันทึกเป็น `hello.sh` แล้วรันได้สองแบบ:

```bash
bash hello.sh Dao
# หรือ
chmod u+x hello.sh
./hello.sh Dao
```

- shebang ระบุ interpreter เมื่อเรียกไฟล์ให้ทำงานโดยตรง
- `./` ระบุ path เพราะ directory ปัจจุบันไม่จำเป็นต้องอยู่ใน `PATH`
- `bash file.sh` ทำงานใน Bash อีกตัว; การเปลี่ยนตัวแปรหรือ directory ไม่ย้อนมาเปลี่ยน shell ผู้เรียก
- `source file.sh` หรือ `. file.sh` ทำงานใน shell ปัจจุบัน
- `bash -n file.sh` ตรวจ syntax โดยไม่รัน; `bash -x file.sh` รันพร้อมแสดง trace
- คำสั่งภายในดูด้วย `help` เช่น `help read`, `help for`

คำสั่งทบทวนใน Lecture 9: `jobs`, `fg`, `bg`, `kill`, `cd -` และ `( commands )` ซึ่งสร้าง subshell

การสำรองไฟล์ที่พบในบท: `tar -cf backup.tar dir/` สร้าง archive, `tar -tf backup.tar` ดูรายการ, `tar -xf backup.tar` แตกไฟล์; เติม `z` เช่น `tar -czf backup.tar.gz dir/` เมื่อต้องการ gzip

## 2. ตัวแปร ขอบเขต และ quote

```bash
name="Dao Jai"            # ห้ามเว้นวรรครอบ =
printf '%s\n' "$name"     # หนึ่ง argument แม้ค่ามีช่องว่าง
printf '%s\n' '${name}'   # พิมพ์ ${name} ตรงตัว
printf '%s\n' "${name}_report"
read -r name              # รับข้อความ; -r ไม่ตีความ backslash เป็น escape
unset name                # ถอนตัวแปร
readonly course=Unix      # อ่านได้ เปลี่ยน/ถอนค่าไม่ได้ใน shell นี้
declare -i count=0        # กำหนด attribute จำนวนเต็ม
```

`name=` คือมีตัวแปรแต่เป็นค่าว่าง; `unset name` คือถอนตัวแปร

| รูปแบบ | ความหมาย |
|---|---|
| `v=value` | ตั้งตัวแปรใน shell นี้ |
| `export v=value` | ให้โพรเซสลูกที่เรียกภายหลังรับค่าผ่าน environment |
| `local v=value` | จำกัดตัวแปรไว้ในขอบเขตฟังก์ชันของ Bash |
| `( v=new )` | เปลี่ยนค่าใน subshell ไม่เปลี่ยนค่าตัวแปรใน shell แม่ |
| `{ v=new; }` | ทำใน shell ปัจจุบัน ค่าจึงเปลี่ยนตาม |

**export ไม่ได้ทำให้แม่และลูกใช้ตัวแปรร่วมกันแบบแก้ไขถึงกันได้** โพรเซสลูกได้รับค่าเริ่มต้น และการเปลี่ยนในลูกไม่ย้อนกลับไปแก้ environment ของแม่

### ค่าว่างและค่าเริ่มต้น

| รูปแบบ | เมื่อ v ว่างหรือยังไม่ถูกตั้ง |
|---|---|
| `${v:-default}` | ใช้ default แทนในการขยาย แต่ไม่กำหนดกลับให้ v |
| `${v:=default}` | กำหนด default ให้ v แล้วใช้ค่านั้น |
| `${v:?message}` | แสดง error; ใน non-interactive shell จะยุติ shell หากเกิด error นี้ |

เครื่องหมาย `:` ทำให้ตรวจทั้ง unset และค่าว่าง; แบบไม่มี `:` เช่น `${v-default}` ตรวจเฉพาะ unset

```bash
unset folder
printf '%s\n' "${folder:-./data}"  # ./data; folder ยัง unset
: "${folder:=./data}"             # folder กลายเป็น ./data
```

## 3. Arguments และ special parameters

| ตัวแปร | ความหมาย |
|---|---|
| `$0` | ชื่อ/เส้นทางที่ใช้เรียก script |
| `$1`, `$2`, … | argument ตามลำดับ |
| `${10}` | argument ลำดับ 10; `$10` ไม่ได้หมายถึงตัวที่ 10 |
| `$#` | จำนวน positional arguments |
| `"$@"` | arguments ทุกตัว โดยคงแต่ละตัวแยกกัน |
| `"$*"` | arguments ทั้งหมดเป็นคำเดียว คั่นด้วยตัวแรกของ IFS |
| `$?` | exit status ของคำสั่งล่าสุด |
| `$$` | process ID ของ shell ตามความหมายของ Bash |
| `$!` | PID ของงานที่เริ่มแบบ background ล่าสุด |

เรียก `./show.sh "red blue" green` → `$#` เป็น 2, `$1` เป็น `red blue`, `$2` เป็น `green`

```bash
for item in "$@"; do
    printf '<%s>\n' "$item"
done
# <red blue>
# <green>
```

`shift` เลื่อน positional parameters: `$2` เดิมมาเป็น `$1` และ `$#` ลดลงหนึ่ง; `shift 2` เลื่อนสองตำแหน่ง ต้องมี arguments เพียงพอ

**อย่าใช้ `$@` แบบไม่ quote เมื่อจำเป็นต้องรักษาช่องว่างใน argument**

## 4. คำนวณ: string กับ arithmetic ต่างกัน

```bash
n=7+2
printf '%s\n' "$n"           # 7+2
n=$((7+2))
printf '%s\n' "$n"           # 9
printf '%s\n' "$((7/2))"     # 3: หารจำนวนเต็ม
printf '%s\n' "$((7%2))"     # 1: เศษ
((n += 1))                   # เปลี่ยนค่า n
```

| รูปแบบ | ใช้ทำอะไร |
|---|---|
| `$(command)` | รับ stdout ของคำสั่ง โดยตัด newline ที่ท้ายผลลัพธ์ |
| `$((expression))` | คำนวณแล้วขยายเป็นข้อความผลลัพธ์ |
| `((expression))` | คำสั่งคำนวณ; status 0 เมื่อนิพจน์ไม่เป็น 0, status 1 เมื่อค่าเป็น 0 |

`^` ใน Bash arithmetic คือ XOR; ยกกำลังใช้ `**` หารทศนิยมให้ใช้เครื่องมือเช่น `awk` หรือ `bc` ที่ติดตั้งไว้

```bash
awk 'BEGIN {printf "%.2f\n", 7/2}'   # 3.50
printf 'scale=2; 7/2\n' | bc        # 3.50
```

ตัวเลขขึ้นต้นด้วย `0` ใน Bash arithmetic อาจถูกมองเป็นฐานแปด เช่น `08` ใช้ไม่ได้ หากรับจำนวนฐานสิบที่มีศูนย์นำหน้าควรตรวจรูปแบบและแปลงฐานให้ชัดเจน

## 5. Exit status, test และ if

**คำสั่งสำเร็จ = status 0; ไม่สำเร็จ/เงื่อนไขเท็จ = nonzero**; `if` ตรวจ status ของคำสั่ง ไม่ใช่อ่านคำว่า true/false ที่พิมพ์ออกมา

```bash
if grep -q 'ERROR' app.log; then
    printf 'Found ERROR\n'
fi
```

`grep` ให้ 0 เมื่อพบ, 1 เมื่อไม่พบ, ส่วน error เป็นค่าอื่น จึงไม่ควรตีความ nonzero ทุกกรณีว่าไม่พบอย่างเดียว

`$?` เปลี่ยนทุกครั้งที่มีคำสั่งใหม่ หากต้องใช้ภายหลังให้บันทึกทันที:

```bash
some_command
status=$?
printf 'status=%s\n' "$status"
```

### ตาราง test ที่ต้องจำ

| กลุ่ม | ตัวดำเนินการ |
|---|---|
| จำนวนเต็ม | `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge` |
| String | `=`, `!=`, `-z` ว่าง, `-n` ไม่ว่าง |
| File | `-e` มีอยู่, `-f` ไฟล์ทั่วไป, `-d` directory, `-s` มีอยู่และขนาดมากกว่า 0 |
| สิทธิ | `-r` อ่านได้, `-w` เขียนได้, `-x` execute/search ได้ |
| เปรียบเทียบไฟล์ | `a -nt b` ใหม่กว่า, `a -ot b` เก่ากว่า, `a -ef b` อ้างไฟล์เดียวกัน |

```bash
if [ "$score" -ge 80 ]; then
    printf 'A\n'
elif [ "$score" -ge 70 ]; then
    printf 'B\n'
else
    printf 'Other\n'
fi
```

สมมติ score ผ่านการตรวจว่าเป็นจำนวนเต็มแล้ว; **ต้องเว้นวรรคหลัง `[` และก่อน `]`** เพราะ `[` เป็นคำสั่ง และ `]` เป็น argument ปิดท้าย

```bash
[ "$n" -ge 1 ] && [ "$n" -le 10 ]  # AND
[ "$x" = yes ] || [ "$x" = y ]     # OR
! [ -f "$file" ]                    # NOT
```

`a && b` ทำ b เมื่อ a สำเร็จ; `a || b` ทำ b เมื่อ a ไม่สำเร็จ; `a; b` ทำ b ต่อโดยไม่ใช้ status ของ a ตัดสิน

### แยกวงเล็บให้ได้

| รูปแบบ | เหมาะกับ |
|---|---|
| `[ "$a" -gt "$b" ]` | เปรียบเทียบจำนวนเต็มแบบ test |
| `[[ "$name" == A* ]]` | Bash conditional จับ pattern ขึ้นต้น A |
| `[[ "$name" == "A*" ]]` | เทียบกับข้อความ A* ตรงตัว |
| `(( a > b ))` | เปรียบเทียบ/คำนวณจำนวนเต็มใน Bash |
| `( commands )` | สร้าง subshell |

`[ "$a" > "$b" ]` มีปัญหา เพราะ shell อาจมอง `>` เป็น redirection; สำหรับ string ใน Bash ใช้ `[[ "$a" > "$b" ]]` ส่วนตัวเลขใช้ `-gt` หรือ `(( ))`

## 6. ลูป for, while, until

### for: ไล่รายการ

```bash
for value in 2 4 6; do
    printf '%s\n' "$value"
done

for arg in "$@"; do
    printf '%s\n' "$arg"
done

for ((i=1; i<=3; i++)); do
    printf '%s\n' "$i"
done
```

`for arg; do ...; done` เป็นรูปย่อของการวน positional arguments แบบรักษาแต่ละ argument

### while: ทำต่อเมื่อเงื่อนไขสำเร็จ

```bash
i=1
while [ "$i" -le 3 ]; do
    printf '%s\n' "$i"
    i=$((i+1))
done
```

### until: ทำต่อเมื่อเงื่อนไขยังไม่สำเร็จ

```bash
i=1
until [ "$i" -gt 3 ]; do
    printf '%s\n' "$i"
    i=$((i+1))
done
```

ทั้งสองตัวอย่างพิมพ์ 1, 2, 3; ตรวจเงื่อนไขก่อนทำ จึงอาจไม่เข้าลูปเลย

- `break` ออกจากลูป
- `continue` ข้ามส่วนที่เหลือของรอบปัจจุบันแล้วเริ่มรอบถัดไป
- ต้องตรวจจุดเริ่มต้น เงื่อนไขหยุด และการอัปเดตตัวแปร ป้องกันวนไม่จบและ off-by-one

### อ่านไฟล์ทีละบรรทัด

```bash
count=0
while IFS= read -r line || [ -n "$line" ]; do
    count=$((count+1))
done < input.txt
printf '%s\n' "$count"
```

`IFS=` รักษาช่องว่างหัว/ท้าย, `-r` รักษา backslash, เงื่อนไขหลัง `||` ช่วยรับบรรทัดท้ายที่ไม่มี newline

`cat input.txt | while ...` มักทำให้ลูปอยู่ใน subshell ของ Bash ค่าตัวแปรที่แก้ในลูปจึงอาจไม่คงอยู่ภายนอก ใช้ redirect เข้า loop ตามตัวอย่างเมื่อจำเป็นต้องใช้ค่าต่อ

## 7. case และ here document

```bash
case "$1" in
    start|run) printf 'Start\n' ;;
    stop)      printf 'Stop\n' ;;
    *.txt)     printf 'Text file\n' ;;
    *)         printf 'Unknown\n' ;;
esac
```

case ใช้ shell patterns เช่น `*`, `?`, `[...]`; เลือกแขนแรกที่ตรง และ `;;` จบแขนนั้น ไม่ใช่ regex

### Here document: ป้อนข้อความหลายบรรทัดเป็น stdin

```bash
name=Dao
cat <<EOF
Hello $name
Today: $(date +%F)
EOF
```

delimiter `EOF` บรรทัดปิดต้องอยู่ตามรูปแบบที่ถูกต้อง ไม่มีช่องว่างเกิน; `<<'EOF'` จะไม่ขยายตัวแปร/คำสั่งในเนื้อหา และ `<<-EOF` ตัด tab นำหน้าได้ แต่ไม่ตัด space

## 8. Function: รับ argument และคืนผล

```bash
sum() {
    local a=$1 b=$2
    printf '%s\n' "$((a+b))"
}
answer=$(sum 4 7)
printf '%s\n' "$answer"       # 11
```

- เรียก `sum 4 7` โดยไม่ใส่วงเล็บแบบ C/Python
- ภายในฟังก์ชัน `$1`, `$2`, `$#` หมายถึง arguments ของฟังก์ชัน
- ตัวแปรที่ไม่ได้ประกาศ `local` อาจแก้ค่าตัวแปรภายนอกได้
- `return 0` คืน **สถานะ** สำเร็จ ไม่ได้คืนค่าข้อมูลทั่วไป; status อยู่ในช่วง 0–255
- ส่งข้อมูลทาง stdout ด้วย `printf` แล้วรับด้วย `$(...)`
- `return` จบฟังก์ชัน; `exit` จบ shell ที่กำลังรัน script

### ตัวอย่างแต่งเพิ่ม: factorial สำหรับจำนวน 0–20

ใช้ช่วงนี้เพื่อให้ผลอยู่ในขอบเขต signed 64-bit ของ Bash ทั่วไป

```bash
#!/usr/bin/env bash
fac() {
    local n=$1 result=1 i
    for ((i=2; i<=n; i++)); do
        result=$((result*i))
    done
    printf '%s\n' "$result"
}

if [ "$#" -eq 0 ]; then
    printf 'Usage: %s integer...\n' "$0" >&2
    exit 1
fi

for n in "$@"; do
    if [[ ! "$n" =~ ^([0-9]|1[0-9]|20)$ ]]; then
        printf 'Expected integer 0..20: %s\n' "$n" >&2
        exit 1
    fi
    fac "$n"
done
```

`./fac.sh 5 0` → 120 และ 1 คนละบรรทัด; `0! = 1` จึงเริ่ม result ที่ 1 และไม่เข้าลูปเมื่อ n=0

## 9. Array ใน Bash

```bash
scores=(65 82 74)
scores[1]=90
printf '%s\n' "${scores[0]}"    # 65
printf '%s\n' "${#scores[@]}"   # 3: จำนวนสมาชิก
printf '%s\n' "${!scores[@]}"   # index แต่ละตัว

for s in "${scores[@]}"; do
    printf '%s\n' "$s"
done

unset 'scores[1]'
```

- indexed array เริ่ม index 0 และมีช่องว่างของ index ได้
- `"${arr[@]}"` ขยายแต่ละสมาชิกเป็นคนละ argument
- `"${arr[*]}"` รวมเป็น argument เดียวด้วยตัวแรกของ IFS
- `${#arr[@]}` = จำนวนสมาชิก; `${#arr[0]}` = ความยาวข้อความสมาชิกแรก
- หลัง unset จำนวนสมาชิกไม่จำเป็นต้องเท่ากับ index สูงสุด + 1; วน index ด้วย `"${!arr[@]}"` เมื่อต้องการรองรับช่องว่าง

```bash
scores=(10 20 30)
sum=0
for s in "${scores[@]}"; do
    sum=$((sum+s))
done
awk -v s="$sum" -v n="${#scores[@]}" \
    'BEGIN {if (n>0) printf "avg=%.2f\n", s/n}'
# avg=20.00
```

## 10. Regex: แยกจาก glob

| รูปแบบ | Glob ที่ shell ใช้เลือกชื่อ | Regex ที่ grep/sed/awk ใช้ค้นข้อความ |
|---|---|---|
| `*` | อักขระอะไรก็ได้ 0 ตัวขึ้นไป | ทำ item ก่อนหน้าซ้ำ 0 ครั้งขึ้นไป |
| `?` | อักขระหนึ่งตัว | ใน ERE ทำ item ก่อนหน้าเป็น optional |
| `.` | จุดตรงตัว | อักขระหนึ่งตัว |
| `^`, `$` | โดยทั่วไปเป็นตัวอักษรธรรมดา | ต้น/ท้ายบรรทัด |

ใช้ `grep -E` สำหรับ ERE: `+` = อย่างน้อยหนึ่ง, `?` = ศูนย์หรือหนึ่ง, `(...)` = กลุ่ม, `a|b` = ทางเลือก, `{m,n}` = จำนวนซ้ำ

```bash
grep -E '^[0-9]+$' data.txt   # ทั้งบรรทัดเป็นเลขอย่างน้อยหนึ่งหลัก
grep -E '^A.*z$' data.txt     # เริ่ม A และจบ z
grep -F 'a.b' data.txt        # ค้นข้อความ a.b แบบตรงตัว
```

`[[:digit:]]` คือ character class สำหรับเลข; `[^0-9]` คือหนึ่งตัวที่ไม่ใช่เลข; `\.` ใช้จุดตรงตัวใน regex

ใส่ single quote รอบ regex เพื่อให้ shell ส่ง pattern ให้เครื่องมือ ส่วน Bash `[[ "$x" =~ regex ]]` ใช้ ERE และการ quote RHS ทั้งหมดจะทำให้ส่วนนั้นถูกเทียบแบบ literal

## 11. sed: แก้ไขข้อความเป็น stream

```bash
sed 's/cat/dog/' input.txt     # เปลี่ยนครั้งแรกของแต่ละบรรทัด
sed 's/cat/dog/g' input.txt    # เปลี่ยนทุกครั้งในแต่ละบรรทัด
sed '2,4d' input.txt           # ตัดบรรทัด 2–4 ออกจาก output
sed '/^#/d' input.txt          # ตัดบรรทัดที่เริ่มด้วย #
sed '/^$/d' input.txt          # ตัดบรรทัดว่างจริง ๆ
sed -n '2,4p' input.txt        # พิมพ์เฉพาะบรรทัด 2–4
sed -E 's/[0-9]+/NUM/g' input.txt
```

โครงสร้าง substitution: `s / pattern / replacement / flags`

- ปกติ sed พิมพ์ผลแต่ละบรรทัดอัตโนมัติ; `-n` ปิดการพิมพ์อัตโนมัติ
- `p` เพิ่มการพิมพ์ จึงอาจเห็นบรรทัดซ้ำเมื่อไม่ใช้ `-n`
- `d` ตัดทั้งบรรทัดจาก output; `s/pattern//` ลบเฉพาะส่วนที่ตรง
- ไม่ใส่ `-i` จะไม่แก้ไฟล์ต้นฉบับ; อย่าเขียน `sed ... file > file` เพราะ shell ตัดไฟล์ก่อน sed อ่าน
- `^$` ไม่ตรงกับบรรทัดที่มีช่องว่าง; ใช้ `^[[:space:]]*$` เมื่อต้องการรวมกรณีนั้น

## 12. awk: ทำงานตาม record และ field

รูปแบบ: `awk 'pattern { action }' file` โดยค่าปกติหนึ่งบรรทัดเป็นหนึ่ง record และแบ่ง field ด้วย whitespace

| ชื่อ | ความหมาย |
|---|---|
| `$0` | record ทั้งหมด |
| `$1`, `$2` | field ที่ 1, 2 |
| `NR` | จำนวน record ที่อ่านแล้วรวมทุกไฟล์ |
| `NF` | จำนวน field ของ record ปัจจุบัน |
| `FS` / `-F` | ตัวคั่น field ขาเข้า |
| `OFS` | ตัวคั่นระหว่างค่าที่ `print` คั่นด้วย comma |
| `BEGIN` / `END` | ทำก่อนอ่านข้อมูล / หลังอ่านข้อมูล |

สมมติ `scores.txt` มี:

```text
Dao 80
Mek 65
Fon 95
```

```bash
awk '{print $1}' scores.txt
# Dao, Mek, Fon คนละบรรทัด

awk '$2 >= 80 {print $1, $2}' scores.txt
# Dao 80
# Fon 95

awk '{sum += $2} END {if (NR>0) printf "%.2f\n", sum/NR}' scores.txt
# 80.00

awk -F: '{print $1, $3}' /etc/passwd
# ชื่อบัญชี กับ UID
```

`$1` ของ awk คือ field ของข้อมูล ไม่ใช่ argument ของ shell; single quote รอบโปรแกรมป้องกัน shell ขยาย `$1` ก่อนส่งให้ awk เมื่อต้องส่งค่าจาก shell ใช้ `awk -v limit="$limit" '$2 >= limit' file`

`print $1,$2` มี OFS คั่น; `print $1 $2` ต่อข้อความกัน การใช้ `-F,` เหมาะกับข้อมูล comma แบบง่าย ไม่ใช่ตัวอ่าน CSV เต็มรูปแบบที่รองรับ quoted comma ทุกกรณี

## 13. โจทย์ประยุกต์แต่งเพิ่ม: นับคำในข้อความภาษาอังกฤษ

ตัวอย่างนี้นิยามคำเป็นช่วงตัวอักษร a–z และใช้สัญลักษณ์อื่นเป็นตัวแบ่งคำ

```bash
export LC_ALL=C
tr '[:upper:]' '[:lower:]' < input.txt |
    tr -cs 'a-z' '\n' |
    sed '/^$/d' |
    sort |
    uniq -c |
    sort -k1,1nr -k2,2 |
    awk '{print $2, $1}' > output.txt
```

อ่านทีละขั้น: ทำตัวเล็ก → หนึ่งคำต่อบรรทัด → ตัดบรรทัดว่าง → เรียงให้คำซ้ำติดกัน → นับ → เรียงความถี่จากมากไปน้อยและเรียงคำเมื่อจำนวนเท่ากัน → สลับเป็น `คำ จำนวน`

เช่น input `Red blue red. Blue green!` → output:

```text
blue 2
red 2
green 1
```

`tr -cs` เปลี่ยนชุดอักขระที่ไม่ใช่ a–z เป็น newline และยุบตัวแบ่งซ้ำ ตัวอย่างนี้จึงไม่ใช้กับการตัดคำไทยโดยตรง

## 14. รายละเอียดเสริมจาก Lecture 15: File descriptor

ทบทวน 0 stdin, 1 stdout, 2 stderr และการเปิด descriptor เพิ่ม:

```bash
exec 3< input.txt       # เปิดอ่านด้วย fd 3
IFS= read -r line <&3
printf '%s\n' "$line"
exec 3<&-              # ปิด fd 3

exec 4> output.txt     # เปิดเขียนด้วย fd 4
printf 'hello\n' >&4
exec 4>&-              # ปิด fd 4
```

`>&4` ส่ง output ไป descriptor ที่เปิดอยู่; `2>&1` ทำให้ fd 2 อ้างปลายทางปัจจุบันของ fd 1 ส่วน `exec` แบบมีเพียง redirection เปลี่ยน descriptor ของ shell ปัจจุบัน

## 15. การบ้านในเว็บบอกทักษะที่ควรซ้อม

**เพิ่มการวิเคราะห์ไฟล์ที่ผู้เรียนให้ไว้ในโฟลเดอร์ `homework final`:** มี hw5 เรื่อง redirection, hw6 รับข้อมูลนิสิต, hw7 calculator, Homework_5_2 เรื่อง function, HW_Array เรื่อง grading และ Lect11 ซึ่งมี workshop เมนู/นับเลขคู่ อ่านรายละเอียดใน [บทวิเคราะห์ฉบับเพิ่ม](../assignments/final-homework-analysis.md) โดยเฉพาะการนับ 8 arguments จากคำสั่งที่ไม่ quote ชื่อและวันเกิด รวมถึงช่วงเกรดที่ตัวอย่างในเอกสารไม่ตรงสูตร

รายการด้านล่างเป็นการบ้านจากเว็บที่ตรวจไว้ก่อนหน้า ไม่ใช่รายชื่อไฟล์ชุดเดียวกับโฟลเดอร์ที่เพิ่มมาครั้งนี้

รายการนี้คือหัวข้องานที่พบจริง ไม่ใช่การคาดการณ์ข้อสอบ

| งานในเว็บ | ทักษะที่ควรทำได้ |
|---|---|
| เวลาแข่งขันเฉลี่ย / แสดง directory เป็น tree / bubble sort | รับข้อมูล ลูป เงื่อนไข array และการอ่านเขียนไฟล์ |
| factorial / prime / แปลง Celsius–Fahrenheit | function, arguments, validation, เลขทศนิยม |
| grading ด้วย array | รวมคะแนน เฉลี่ย variance และ SD |
| นับความถี่คำ / ประมวลบทละคร | pipeline, regex, grep/sed/awk, sort/uniq |

สำหรับ array grading สูตรที่โจทย์ระบุใช้ population variance: `avg=Σx/N`, `var=Σ(x−avg)²/N`, `sd=√var` ให้ใช้สูตรและช่วงเกรดที่โจทย์กำหนด อ่านเครื่องหมาย `<`/`<=` ให้ตรง

**ข้อสังเกตจากการตรวจเอกสาร:** ตัวอย่างเฉลยบางส่วนไม่สอดคล้องกับสูตร เช่นเวลา `12.2, 11.8, 12.5, 10.9, 11.1` เฉลี่ยได้ **11.7**; ตัวอย่าง grading คะแนน 1–10 มี avg=5.5 และ SD≈2.87228 ดังนั้นการแจกเกรดตามสูตรที่พิมพ์ไว้จะไม่ตรงกับรายการเกรดตัวอย่างทั้งหมด ใช้เป็นโจทย์ตรวจเหตุผล และยึดคำชี้แจงอาจารย์เมื่อกำหนดคำตอบส่ง

## 16. แบบฝึกหัดแต่งเพิ่มพร้อมเฉลย

1. `x=7; echo '$x'; echo "$x"` → **`$x` แล้ว `7`**
2. `echo "$((9/4))"` → **2** เพราะ integer division
3. `[ 8 -gt 2 ]; echo $?` → **0**
4. `((0)); echo $?` → **1**; `((5)); echo $?` → **0**
5. `./x.sh "a b" c` มี `$#` เท่าไร → **2**
6. หลัง `shift` จากข้อ 5 `$1` เหลืออะไร → **c**
7. `for x in "$@"` ต่างจาก `for x in "$*"` อย่างไร → **แบบแรกวนทีละ argument; แบบหลังวนข้อความรวมหนึ่งรอบ**
8. `i=0; until [ "$i" -ge 3 ]; do echo "$i"; i=$((i+1)); done` → **0, 1, 2 คนละบรรทัด**
9. `a=(x y z); unset 'a[1]'; echo "${#a[@]}"` → **2**
10. ใน array ข้อ 9 index ใดเหลืออยู่ → **0 และ 2**
11. `printf 'cat cat\n' | sed 's/cat/dog/'` → **dog cat**
12. เปลี่ยน sed ข้อ 11 เป็น `s/cat/dog/g` → **dog dog**
13. `printf 'A:10:B\n' | awk -F: '{print NF, $2}'` → **3 10**
14. ต้องการฟังก์ชันส่งค่าผลรวม 300 ควร `return 300` หรือไม่ → **ไม่ควร ใช้ stdout; return เป็น exit status**
15. `${v:-x}` กับ `${v:=x}` ต่างกันอย่างไร → **แบบหลังตั้งค่าให้ตัวแปรเมื่อว่าง/unset; แบบแรกไม่ตั้ง**
16. ถ้า `false; echo ready; echo $?` จะได้ status ของ false หรือไม่ → **ไม่ได้ จะได้ 0 ของ echo ready เมื่อ echo สำเร็จ**
17. `sed -n '1,2p'` กับ `sed '1,2p'` ต่างกันอย่างไร → **แบบแรกพิมพ์เฉพาะ 1–2; แบบหลังยังพิมพ์ทุกบรรทัดและเพิ่มสำเนา 1–2 อีกครั้ง**
18. `ls *.txt` กับ `grep '.*txt' file` ใช้ pattern ภาษาเดียวกันไหม → **ไม่ใช่ แบบแรก glob ของ shell แบบหลัง regex ของ grep**

## อ่านรอบสุดท้าย

- [ ] เขียน if/elif/else, for, while, until, case และ function จากหน้าว่างได้
- [ ] จำ `0=สำเร็จ`, quote, `"$@"`, `shift` และการตรวจจำนวน arguments ได้
- [ ] แยก `[ ]`, `[[ ]]`, `(( ))`, `$(( ))`, `$( )` และ `( )` ได้
- [ ] รู้ว่า export, local, subshell และ source มีผลต่อค่าตัวแปรอย่างไร
- [ ] วน array โดยไม่สมมติว่า index ต่อเนื่องเสมอ
- [ ] แยก glob/BRE/ERE; ใช้ anchor และ character class ได้
- [ ] อ่านผล sed และ awk ทีละบรรทัด/field ได้
- [ ] ต่อ pipeline กรอง แปลง นับ และเรียงข้อมูลได้
- [ ] ตรวจ input, กรณีไม่มี argument, ค่าว่าง, 0 และขอบเขตลูป

## แหล่งประกอบ

- [โครงร่างรายวิชา](https://angsila.cs.buu.ac.th/~nutthanon/88624159/)
- [Lecture 9: เริ่ม Shell Script](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect09.pdf)
- [Lecture 10: variables, test และ if](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lecture10.pdf)
- [Lecture 11: arguments, if และ for](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect11.pdf)
- [Lecture 12: loop](https://staff.cs.buu.ac.th/~jira/886326/Lectures/week12/Lect12.txt)
- [Lecture 13: case และ here document](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect13.pdf)
- [Lecture 15: function และ file descriptor](https://staff.informatics.buu.ac.th/~nutthanon/88624159/Lect15.pdf)
- [Array: Cornell Lecture 16](https://www.cs.cornell.edu/courses/cs2043/2014sp/lectures/lecture16.pdf)
- [Regex/sed/awk: Brown](https://cs.brown.edu/courses/cs0060/assets/slides/slides9.pdf)
- [การบ้าน loop](https://angsila.cs.buu.ac.th/~nutthanon/88624159/Homework_Lect12.pdf), [function](https://angsila.cs.buu.ac.th/~nutthanon/88624159/Homework_5_2.pdf), [array](https://angsila.cs.buu.ac.th/~nutthanon/88624159/HW_Array.pdf), [text processing](https://angsila.cs.buu.ac.th/~nutthanon/88624159/Homework_7.pdf)

สรุปนี้เรียบเรียงใหม่ พร้อมตัวอย่างแต่งเพิ่ม คำอธิบายเรื่อง environment, pipeline และ parameter expansion ระบุพฤติกรรม Bash ให้ชัดในจุดที่เอกสารเก่าอาจทำให้เข้าใจคลาดเคลื่อน
