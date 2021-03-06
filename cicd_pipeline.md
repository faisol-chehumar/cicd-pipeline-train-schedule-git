# แนะนำวิธีการทำ CI/CD Pipeline
----

## สิ่งที่จะได้รับจากบทความนี้
  - แนะนำ Tools และเทคนิคในการทำ CI/CD บางส่วน
  - เข้าใจขั้นตอนในการนำ CI/CD ไป implement ในงานจริง
  - ความรู้พื้นฐานพอที่จะใช้ในการตัดสินใจเลือกใช้ Tools แบบต่างๆ
  - แนะนำเทคนิคและ tools ที่คุณสามารถนำไปใช้อย่างเช่น CI tools และเทคนิคการทำ automation deploy
  - แนะนำ tools ที่ใช้ในงาน infrastructure เพื่อใช้ในส่วนงาน CD
  ----

## แนะนำการใช้ SCM (Source control management)
SCM เป็นส่วนที่สำคัญในการทำ CI/CD โดยจะเป็นส่วนแรกๆที่เราต้องใช้ สำหรับใครก็ตามที่อยากทำ CI/CD จะต้องเรียนรู้และทำความเข้าใจการใช้ SCM ซะก่อน โดนปัจจุบันเครื่องมือสำหรับทำ SCM ที่นิยมใช้กันมากที่สุดคือ Git โดยบทความนี้จะแสดงให้เห็นถึงความสัมพันธ์ระหว่าง SCM และ CI/CD โดยเราจะเริ่มการที่การติดตั้ง Git เพื่อใช้งานก่อน


### บทบาทสำคัญของ SCM
  - SCM เป็นส่วนหนึ่งของ Workflow ที่ Dev ใช้เป็นประจำทุกๆวัน
    + การเปลี่ยนแปลงของโค้ดที่เกิดจาก developer จะต้องถูก track ได้ผ่านทาง SCM
    + Dev สามารถใช้ SCM ในการ track ส่วนของโค้ดที่เปลี่ยนแปลงและใช้มันในการรวมโค้ดที่แก้ไขเสร็จแล้วเข้าด้วยกันได้
  - ส่วนของ automation ใดๆ ที่จะมีการปฏิสัมพันธ์กับโค้ดจะต้องทำผ่าน SCM
    + ส่วนของ CI จะใช้โค้ดจาก SCM
    + SCM จะส่ง Notify ไปหา CI server เมื่อมีโค้ดที่ต้องการ Build

## GIT SCM
เราจะใช้ Git และ Github ในการจัดการ Source Code
- การจำไปต่อเรามาทำความเข้าใจ Git-base workflow กันก่อน
- คุณจะต้องรู้ขั้นตอนที่จะใช้เพื่อจัดการ Source code โดยใช้ Git

Step ในการใช้ Git พื้นฐานที่ต้องรู้
  - Cloning
  - Adding
  - Commiting
  - Pushing
  - Branching
  - Merging
  - Pull Requests

### ติดตั้ง
  - Download และติดตั้ง Git จากเว็บ [https://git-scm.com/downloads]( https://git-scm.com/downloads)
  - Setting Config ด้วย Name และ Email ตามรูปภาพ [img1]
  - Setting Private key เพื่อใช้งาน ssh บน remote git server ของ github ด้วยคำสั่ง `ssh-keygen -t rsa -b 4096` โดยที่ Private key และ Public key จะถูกสร้างไว้ใน directory `~/.ssh` (คำสั่งนี้ใช้ผ่าน terminal ถ้าใช้ Window ต้องใช้ Putty) ตามคำสั่งในรูปภาพ [img2]
  - เปิดไฟล์ id_rsa.pub แล้ว copy ไว้ ไปที่เว็บ Github > Setting > SSH and GPG keys แล้วนำ public key ที่ copy ไว้ใส่ลงในช่วง Key ส่วน Title ให้ตั้งชื่ออะไรก็ได้ [img3]

### การใช้งาน Fork
Fork เป็นเครื่องมือที่ใช้ในการ Copy โปรเจคที่เราสนใจมาพัฒนาต่อ ซึ่งเราสามารถควบคุมโปรเจคได้เต็มที่โดยไม่กระทบกับโปรเจคต้นฉบับ เราจะเห็นได้บ่อยในการนำมาใช้จัดการโปรเจคที่เป็น Open source เพื่อให้นักพัฒนาทำงานร่วมกันได้ ส่วน Step ที่เราจะใช้คือ เราต้องทำการ Fork โปรเจคที่เราต้องการนำมาพัฒนาต่อ จากนั้นก็ Clone โปรเจคลงมาทำในเครื่องของเรา เมื่อเราพัฒนา Feature ใหม่เสร็จแล้วก็ Push กลับไปที่ Repo ของเรา จากนั้นก็ทำการ Pull Request ไปที่โปรเจคต้นฉบับ เพื่อให้เจ้าของโปรเจคทำการ Code review หากผ่าน เค้าก็จะทำการ Merge ส่วนของ Feature ใหม่ๆ ที่เราพัฒนาเข้าไปในในโปรเจคต้้นฉบับ

### git clone <repo url>
หากคุณกำลังทำงานอยู่บนโปรเจคที่ถูก Create ไว้อยู่แล้ว สิ่งแรกที่คุณต้องทำคือการ Clone โปรเจคมาไว้ในเครื่องงตัวเองผ่านคำสั่ง `git clone <url repo ของโปรเจคที่คุณต้องการ>`
ซึ่งคำสั่ง clone นี้จะทำงานดังนี้
- Download สิ่งที่อยู่ใน repo ทั้งหมดลงมาไว้ในเครื่อง local รวมถึงพวก history ต่างๆ ของโปรเจคด้วย

หากเป็นการสร้างโปรเจคขึ้นมาใหม่ ให้ใช้คำสั่ง `git init` ใน folder โปรเจคที่คุณสร้างขึ้นมาใหม่

### git status
ใช้สำหรับเช็คดูข้อมูลต่างๆของ local repo อย่างเช่น ไฟล์ไหนมีการเปลี่ยนแปลง ไฟล์ไหนยังไม่ได้ถูก commit หรือตอนนี้เรากำลังทำงานอยู่ที่ bracnch ไหน

### git add <file>
คำสั่งนี้ใช้สำหรับการ add ไฟล์ลงใน stages เพื่อรอการ commit ต่อไป โดยที่เราสามารถเลือก Add เฉพาะบางไฟล์หรือ Add ทุกไฟล์ผ่านคำสั่ง `git add .`

### git commit -m "ใส่คอมเม้นในนี้"
คำสั้งนี้ใช้เพื่อใส่รายละเอียดคร่าวๆว่าเราได้เปลี่ยนแปลงโค้ดยังไงบ้าง

### git push
หากเราใช้แค่คำสั่ง commit การเปลี่ยนแปลงโค้ดจะอยู่เพียงระดับ local หรือภายในเครื่องของเราเท่านั้น ดังนั้นถ้าเราต้องการ update โค้ดบน remote git server จะต้องใช้คำสั้ง `git push` ด้วย และค่า default ของคำสั่งนี้จะทำการ push code ไปที่ repo/branch ที่เราทำการ clone มาในตอนแรก หากเราต้องการ push โค้ดโดยที่เราเราสร้างโปรเจคมาใหม่ให้ใช้คำสั่ง `git push -u <repo url> <branch>`