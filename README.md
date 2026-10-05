## Students REST API

### โครงสร้างไฟล์

![โครงสร้างไฟล์](https://bucket.kku.ac.th/iskku/github/Screenshot_2026-10-02_102534.png)

### HTTP Request

![HTTP Request](https://bucket.kku.ac.th/iskku/github/b580cba8-fc0f-440e-b6ca-256bd938fa28.png)

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

export async function GET() {
  const students = await prisma.student.findMany({
    orderBy: {
      id: "asc",
    },
  });

  return NextResponse.json(students);
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

แก้ไขไฟล์
_app/students/create/route.ts_
เพิ่ม

```
import { studentSchema } from "@/app/students/validation";
```

เพิ่ม function

```tsx
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

### Search + Filter + URL State

### Search Form

สร้าง
_app/students/search-form.tsx_

```tsx
type SearchFormProps = {
  search?: string;
  major?: string;
  status?: string;
};

export default function SearchForm({
  search = "",
  major = "",
  status = "",
}: SearchFormProps) {
  return (
    <form
      method="GET"
      className="mb-6 rounded-lg border border-gray-200 bg-white p-4 shadow-sm"
    >
      <div className="grid gap-4 md:grid-cols-4">
        {/* Search */}
        <div className="md:col-span-2">
          <label
            htmlFor="search"
            className="mb-1 block text-sm font-medium text-gray-700"
          >
            ค้นหา
          </label>

          <input
            id="search"
            name="search"
            type="text"
            defaultValue={search}
            placeholder="ค้นหารหัส หรือชื่อ..."
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        {/* Major */}
        <div>
          <label
            htmlFor="major"
            className="mb-1 block text-sm font-medium text-gray-700"
          >
            สาขา
          </label>

          <select
            id="major"
            name="major"
            defaultValue={major}
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          >
            <option value="">ทุกสาขา</option>
            <option value="Information Technology">
              Information Technology
            </option>
            <option value="Computer Science">Computer Science</option>
            <option value="Digital Technology">Digital Technology</option>
          </select>
        </div>

        {/* Status */}
        <div>
          <label
            htmlFor="status"
            className="mb-1 block text-sm font-medium text-gray-700"
          >
            สถานะ
          </label>

          <select
            id="status"
            name="status"
            defaultValue={status}
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          >
            <option value="">ทั้งหมด</option>
            <option value="active">กำลังศึกษา</option>
            <option value="inactive">พ้นสภาพ</option>
          </select>
        </div>
      </div>

      <div className="mt-4 flex gap-2">
        <button
          type="submit"
          className="rounded-md bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700"
        >
          ค้นหา
        </button>

        <a
          href="/students"
          className="rounded-md border border-gray-300 px-4 py-2 font-medium text-gray-700 hover:bg-gray-50"
        >
          ล้าง
        </a>
      </div>
    </form>
  );
}
```

แก้ไขไฟล์
_app/students/page.tsx_

```tsx
import prisma from "@/app/lib/prisma";
import SearchForm from "./search-form";
import DeleteButton from "./delete-button";

type StudentsPageProps = {
  searchParams: Promise<{
    search?: string;
    major?: string;
    status?: string;
  }>;
};

export default async function StudentsPage({
  searchParams,
}: StudentsPageProps) {
  const params = await searchParams;

  const search = params.search ?? "";
  const major = params.major ?? "";
  const status = params.status ?? "";

  const students = await prisma.student.findMany({
    where: {
      AND: [
        search
          ? {
              OR: [
                {
                  studentCode: {
                    contains: search,
                  },
                },
                {
                  name: {
                    contains: search,
                  },
                },
              ],
            }
          : {},

        major
          ? {
              major: major,
            }
          : {},

        status
          ? {
              status: status === "active",
            }
          : {},
      ],
    },

    orderBy: {
      id: "asc",
    },
  });

  return (
    <main className="mx-auto max-w-6xl p-6">
      <div className="mb-6">
        <h1 className="text-3xl font-bold text-gray-900">Student Management</h1>

        <p className="mt-2 text-gray-600">จัดการข้อมูลนักศึกษา</p>
      </div>

      <SearchForm search={search} major={major} status={status} />
      <div className="flex justify-between">
        <div className="mb-4 text-sm text-gray-600">
          พบข้อมูล {students.length} รายการ
        </div>
        <div>
          <a
            href={`/students/create`}
            className="inline-block rounded bg-blue-600 my-5 px-6 py-2.5 text-sm font-medium text-white shadow-md transition duration-150 ease-in-out hover:bg-blue-700 hover:shadow-lg focus:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
          >
            เพิ่มนักศึกษา
          </a>
        </div>
      </div>

      <div className="overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm">
        <table className="w-full">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-4 py-3 text-left">รหัส</th>

              <th className="px-4 py-3 text-left">ชื่อ</th>

              <th className="px-4 py-3 text-left">Email</th>

              <th className="px-4 py-3 text-left">สาขา</th>

              <th className="px-4 py-3 text-left">ชั้นปี</th>

              <th className="px-4 py-3 text-left">สถานะ</th>

              <th className="px-4 py-3">จัดการ</th>
            </tr>
          </thead>

          <tbody className="divide-y divide-gray-200">
            {students.map((student) => (
              <tr key={student.id}>
                <td className="px-4 py-3">{student.studentCode}</td>

                <td className="px-4 py-3">{student.name}</td>

                <td className="px-4 py-3">{student.email ?? "-"}</td>

                <td className="px-4 py-3">{student.major}</td>

                <td className="px-4 py-3">{student.year}</td>

                <td className="px-4 py-3">
                  {student.status ? "กำลังศึกษา" : "พ้นสภาพ"}
                </td>

                <td className="px-4 py-3">
                  <div className="flex gap-2">
                    <a
                      href={`/students/edit/${student.id}`}
                      className="rounded-md bg-yellow-500 px-3 py-1.5 text-sm text-white"
                    >
                      แก้ไข
                    </a>

                    <DeleteButton id={student.id} />
                  </div>
                </td>
              </tr>
            ))}

            {students.length === 0 && (
              <tr>
                <td colSpan={7} className="px-4 py-8 text-center text-gray-500">
                  ไม่พบข้อมูลนักศึกษา
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>
    </main>
  );
}
```

แก้ไขไฟล์

_app/students/create/page.tsx_

```tsx
<select
  id="major"
  name="major"
  className="w-full rounded-md border border-gray-300 px-3 py-2"
>
  <option value="Information Technology">Information Technology</option>
  <option value="Computer Science">Computer Science</option>
  <option value="Digital Technology">Digital Technology</option>
</select>
```

แก้ไขไฟล์
_app/students/edit/[id]/page.tsx_

```tsx
<select
  id="major"
  name="major"
  defaultValue={student.major}
  className="w-full rounded-md border border-gray-300 px-3 py-2"
>
  <option value="Information Technology">Information Technology</option>
  <option value="Computer Science">Computer Science</option>
  <option value="Digital Technology">Digital Technology</option>
</select>
```
