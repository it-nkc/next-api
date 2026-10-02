## Students REST API

### โครงสร้างไฟล์

![โครงสร้างไฟล์](https://bucket.kku.ac.th/iskku/github/Screenshot%202026-10-02%20102534.png)

### สร้าง Validation Schema

เพิ่มไฟล์
_app/students/validation.ts_

```tsx
import { z } from "zod";

export const studentSchema = z.object({
  studentCode: z
    .string()
    .min(1, "กรุณากรอกรหัสนักศึกษา")
    .max(20, "รหัสนักศึกษาต้องไม่เกิน 20 ตัวอักษร"),

  name: z
    .string()
    .min(1, "กรุณากรอกชื่อ-นามสกุล")
    .max(100, "ชื่อ-นามสกุลต้องไม่เกิน 100 ตัวอักษร"),

  email: z.string().email("รูปแบบ Email ไม่ถูกต้อง").or(z.literal("")),

  major: z
    .string()
    .min(1, "กรุณากรอกสาขา")
    .max(100, "ชื่อสาขาต้องไม่เกิน 100 ตัวอักษร"),

  year: z
    .number()
    .int("ชั้นปีต้องเป็นจำนวนเต็ม")
    .min(1, "ชั้นปีต้องไม่น้อยกว่า 1")
    .max(8, "ชั้นปีต้องไม่มากกว่า 8"),
});
```
