## Students REST API

### โครงสร้างไฟล์

![โครงสร้างไฟล์](https://bucket.kku.ac.th/iskku/github/Screenshot_2026-10-02_102534.png)

### HTTP Request

![HTTP Request](https://bucket.kku.ac.th/iskku/github/b580cba8-fc0f-440e-b6ca-256bd938fa28.png)

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

### API สำหรับ GET + POST

สร้างไฟล์:
_app/api/students/route.ts_

```tsx
import prisma from "@/app/lib/prisma";
import { NextResponse } from "next/server";
import { studentSchema } from "@/app/students/validation";

export async function GET() {
  const students = await prisma.student.findMany({
    orderBy: {
      id: "asc",
    },
  });

  return NextResponse.json(students);
}

export async function POST(request: Request) {
  try {
    const body = await request.json();

    const result = studentSchema.safeParse({
      studentCode: body.studentCode,
      name: body.name,
      email: body.email ?? "",
      major: body.major,
      year: Number(body.year),
    });

    if (!result.success) {
      return NextResponse.json(
        {
          message: "ข้อมูลไม่ถูกต้อง",
          errors: result.error.flatten().fieldErrors,
        },
        {
          status: 400,
        },
      );
    }

    const student = await prisma.student.create({
      data: result.data,
    });

    return NextResponse.json(student, {
      status: 201,
    });
  } catch {
    return NextResponse.json(
      {
        message: "ไม่สามารถสร้างข้อมูลนักศึกษาได้",
      },
      {
        status: 500,
      },
    );
  }
}
```

### ทดลอง GET

เปิด Browser หรือ postman

```
http://localhost:3000/api/students
```

```json
[
  {
    "id": 1,
    "studentCode": "66010001",
    "name": "สมชาย ใจดี",
    "email": "somchai@example.com",
    "major": "Information Technology",
    "year": 3,
    "status": true,
    "createdAt": "...",
    "updatedAt": "..."
  }
]
```

### GET นักศึกษารายคน

สร้าง:

_app/api/students/[id]/route.ts_

```tsx
import prisma from "@/app/lib/prisma";
import { NextResponse } from "next/server";

type RouteContext = {
  params: Promise<{
    id: string;
  }>;
};

export async function GET(request: Request, { params }: RouteContext) {
  const { id } = await params;

  const studentId = Number(id);

  if (!Number.isInteger(studentId)) {
    return NextResponse.json(
      {
        message: "Student ID ไม่ถูกต้อง",
      },
      {
        status: 400,
      },
    );
  }

  const student = await prisma.student.findUnique({
    where: {
      id: studentId,
    },
  });

  if (!student) {
    return NextResponse.json(
      {
        message: "ไม่พบข้อมูลนักศึกษา",
      },
      {
        status: 404,
      },
    );
  }

  return NextResponse.json(student);
}
```

ทดลอง get student by id

```
http://localhost:3000/api/students/1
```

```json
{
  "id": 1,
  "studentCode": "66010001",
  "name": "สมชาย ใจดี",
  ...
}
```

ถ้าไม่มี

```
/api/students/999
```

```json
{
  "message": "ไม่พบข้อมูลนักศึกษา"
}
```

### POST เพิ่มนักศึกษา

ใช้ Postman

```
POST /api/students
Content-Type: application/json
```

Body:

```json
{
  "studentCode": "66010010",
  "name": "ทดสอบ API",
  "email": "api@example.com",
  "major": "Computer Science",
  "year": 2
}
```

Response จะเป็น

```json
{
  "id": 10,
  "studentCode": "66010010",
  "name": "ทดสอบ API",
  "email": "api@example.com",
  "major": "Computer Science",
  "year": 2,
  "status": true
}
```

### PUT แก้ไขข้อมูล

แก้ไขไฟล์
_app/api/students/[id]/route.ts_

เพิ่ม

```tsx
import { studentSchema } from "@/app/students/validation";
```

เพิ่ม function

```tsx
export async function PUT(request: Request, { params }: RouteContext) {
  try {
    const { id } = await params;

    const studentId = Number(id);

    if (!Number.isInteger(studentId)) {
      return NextResponse.json(
        {
          message: "Student ID ไม่ถูกต้อง",
        },
        {
          status: 400,
        },
      );
    }

    const body = await request.json();

    const result = studentSchema.safeParse({
      studentCode: body.studentCode,
      name: body.name,
      email: body.email ?? "",
      major: body.major,
      year: Number(body.year),
    });

    if (!result.success) {
      return NextResponse.json(
        {
          message: "ข้อมูลไม่ถูกต้อง",
          errors: result.error.flatten().fieldErrors,
        },
        {
          status: 400,
        },
      );
    }

    const student = await prisma.student.update({
      where: {
        id: studentId,
      },
      data: result.data,
    });

    return NextResponse.json(student);
  } catch {
    return NextResponse.json(
      {
        message: "ไม่สามารถแก้ไขข้อมูลนักศึกษาได้",
      },
      {
        status: 500,
      },
    );
  }
}
```

Body:

```json
{
  "studentCode": "66010010",
  "name": "ทดสอบ API UPDATE",
  "email": "update@example.com",
  "major": "Digital Technology",
  "year": 3
}
```

### DELETE

แก้ไฟล์
_app/api/students/[id]/route.ts_

```tsx
export async function DELETE(request: Request, { params }: RouteContext) {
  try {
    const { id } = await params;

    const studentId = Number(id);

    if (!Number.isInteger(studentId)) {
      return NextResponse.json(
        {
          message: "Student ID ไม่ถูกต้อง",
        },
        {
          status: 400,
        },
      );
    }

    await prisma.student.delete({
      where: {
        id: studentId,
      },
    });

    return NextResponse.json({
      message: "ลบนักศึกษาเรียบร้อยแล้ว",
    });
  } catch {
    return NextResponse.json(
      {
        message: "ไม่พบข้อมูลนักศึกษาหรือไม่สามารถลบได้",
      },
      {
        status: 404,
      },
    );
  }
}
```

ทดสอบ:

```http
DELETE /api/students/10
```

Response

```json
{
  "message": "ลบนักศึกษาเรียบร้อยแล้ว"
}
```
