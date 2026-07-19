## Next.js API: Password Hashing with bcrypt

### Package Install
```bash
npm install bcryptjs
```
---

### lib/auth/password.ts
```bash
import bcrypt from "bcryptjs";

const SALT_ROUNDS = 12; // production standard — 10-12 recommended

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

export async function verifyPassword(
  hashValue: string,
  password: string
): Promise<boolean> {
  return bcrypt.compare(password, hashValue);
}
```
---

### app/api/auth/signup/route.ts
```bash
import { NextRequest, NextResponse } from "next/server";
import prisma from "@/lib/prisma";
import { log } from "@/lib/logger";
import { handlePrismaError } from "@/lib/errors/handlePrismaError";
import { SignupSchema } from "@/lib/validations/auth.schema";
import { hashPassword } from "@/lib/auth/password";


export async function POST(request: NextRequest) {
    const requestId = crypto.randomUUID();
    try {
        log.info(
            { 
                requestId, 
                path: request.nextUrl.pathname, 
                method: request.method 
            },
            "User data received"
        )

        const jsonBody = await request.json();
        const validation = SignupSchema.safeParse(jsonBody);

        if (!validation.success) {
            log.warn(
                { 
                    requestId, 
                    errors: validation.error.flatten().fieldErrors 
                },
                "Invalid User Input"
            )
            return NextResponse.json(
                { status: "fail", requestId, message: "Invalid user input", errors: validation.error.flatten().fieldErrors },
                { status: 400 }
            )
        }

        const { password, ...userData } = validation.data;
        const passwordHash = await hashPassword(password);


        const result = await prisma.user.create({
            data: {
                ...userData,
                passwordHash,
            },
        })


        log.info(
            { 
                requestId, 
                name: result.name, 
                email: result.email 
            },
            "User created"
        )

        return NextResponse.json(
            {
                status: "success",
                requestId,
                message: "Account created.",
                data: { id: result.id, name: result.name, email: result.email },
            },
            { status: 201 }
        )
    } catch (error) {
        const { status, message } = handlePrismaError(error);
        log.error(
            {
                requestId,
                err: error instanceof Error
                    ? { message: error.message, stack: error.stack, name: error.name }
                    : String(error),
            },
            message
        )
        return NextResponse.json(
            { status: "fail", requestId, message },
            { status }
        )
    }
}
```
---
