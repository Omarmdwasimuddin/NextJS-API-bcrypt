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
