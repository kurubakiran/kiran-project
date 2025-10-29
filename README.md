"use client";

import React, { useState } from "react";
import { z } from "zod";
import { useRouter } from "next/navigation";

const SignInSchema = z.object({
  email: z.string().email({ message: "Please enter a valid email" }),
  password: z.string().min(6, { message: "Password must be at least 6 characters" }),
  remember: z.boolean().optional(),
});

type SignInData = z.infer<typeof SignInSchema>;

function Input({
  id,
  type = "text",
  label,
  value,
  onChange,
  error,
  placeholder,
}: {
  id: string;
  type?: string;
  label?: string;
  value: string | boolean;
  onChange: (v: any) => void;
  error?: string | null;
  placeholder?: string;
}) {
  return (
    <div className="mb-4">
      {label && (
        <label htmlFor={id} className="block text-sm font-medium text-slate-700 mb-1">
          {label}
        </label>
      )}
      <input
        id={id}
        type={type}
        value={String(value)}
        onChange={(e) => onChange(e.target.value)}
        placeholder={placeholder}
        className={
          "w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-slate-400 " +
          (error ? "border-red-500" : "border-slate-200")
        }
        aria-invalid={!!error}
        aria-describedby={error ? `${id}-error` : undefined}
      />
      {error && (
        <p id={`${id}-error`} className="text-xs text-red-600 mt-1">
          {error}
        </p>
      )}
    </div>
  );
}

export default function SignInPage() {
  const router = useRouter();
  const [form, setForm] = useState<SignInData>({ email: "", password: "", remember: false });
  const [errors, setErrors] = useState<Partial<Record<keyof SignInData, string | null>>>({});
  const [loading, setLoading] = useState(false);
  const [serverError, setServerError] = useState<string | null>(null);

  const onChange = (field: keyof SignInData, value: any) => {
    setForm((s) => ({ ...s, [field]: value }));
    setErrors((e) => ({ ...e, [field]: null }));
    setServerError(null);
  };

  const validate = () => {
    const result = SignInSchema.safeParse(form);
    if (!result.success) {
      const zErrors: Partial<Record<keyof SignInData, string>> = {};
      for (const err of result.error.errors) {
        const path = err.path[0] as keyof SignInData | undefined;
        if (path) zErrors[path] = err.message;
      }
      setErrors(zErrors);
      return false;
    }
    setErrors({});
    return true;
  };

  const onSubmit = async (e?: React.FormEvent) => {
    e?.preventDefault();
    if (!validate()) return;

    setLoading(true);
    setServerError(null);
    try {
      const res = await fetch("/api/auth/mock-signin", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form),
      });

      const json = await res.json();
      if (!res.ok) {
        setServerError(json?.message || "Sign in failed");
        setLoading(false);
        return;
      }

      // On success: navigate to dashboard or home
      // If you have a real auth flow, set cookies / tokens appropriately
      router.push("/dashboard"); // change to your post-login route
    } catch (err: any) {
      setServerError(err?.message || "Network error");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-slate-50 p-6">
      <div className="w-full max-w-md bg-white border border-slate-200 rounded-2xl p-6 shadow">
        <div className="mb-6 text-center">
          <h1 className="text-2xl font-semibold text-slate-900">Sign in to Blogify</h1>
          <p className="text-sm text-slate-500 mt-1">Enter your credentials to continue</p>
        </div>

        {serverError && (
          <div className="mb-4 text-sm text-red-700 bg-red-50 border border-red-100 p-3 rounded">
            {serverError}
          </div>
        )}

        <form onSubmit={onSubmit} noValidate>
          <Input
            id="email"
            type="email"
            label="Email"
            value={form.email}
            onChange={(v) => onChange("email", v)}
            error={errors.email ?? null}
            placeholder="you@example.com"
          />

          <Input
            id="password"
            type="password"
            label="Password"
            value={form.password}
            onChange={(v) => onChange("password", v)}
            error={errors.password ?? null}
            placeholder="Enter your password"
          />

          <div className="flex items-center justify-between mb-4">
            <label className="flex items-center gap-2 text-sm text-slate-700">
              <input
                type="checkbox"
                checked={form.remember}
                onChange={(e) => onChange("remember", e.target.checked)}
                className="rounded border-slate-300 text-slate-700 focus:ring-0"
              />
              Remember me
            </label>

            <a className="text-sm text-slate-600 hover:underline" href="#">
              Forgot password?
            </a>
          </div>

          <button
            type="submit"
            disabled={loading}
            className={
              "w-full inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md font-medium " +
              (loading ? "bg-slate-300 text-slate-700" : "bg-slate-900 text-white hover:bg-slate-800")
            }
          >
            {loading ? "Signing in..." : "Sign in"}
          </button>
        </form>

        <div className="mt-6 text-center text-sm text-slate-600">
          Don't have an account?{" "}
          <a href="/auth/signup" className="text-slate-900 font-medium hover:underline">
            Create one
          </a>
        </div>

        <div className="mt-6">
          <div className="relative">
            <div className="absolute inset-0 flex items-center">
              <div className="w-full border-t border-slate-100" />
            </div>
            <div className="relative flex justify-center text-xs">
              <span className="bg-white px-2 text-slate-400">or continue with</span>
            </div>
          </div>

          <div className="mt-4 grid gap-2">
            <button
              onClick={() => alert("Social sign-in placeholder")}
              className="w-full inline-flex items-center justify-center gap-2 px-4 py-2 border rounded-md hover:bg-slate-50"
            >
              Continue with Google
            </button>
            <button
              onClick={() => alert("Social sign-in placeholder")}
              className="w-full inline-flex items-center justify-center gap-2 px-4 py-2 border rounded-md hover:bg-slate-50"
            >
              Continue with GitHub
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
import { NextResponse } from "next/server";
import { z } from "zod";

const BodySchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
  remember: z.boolean().optional(),
});

export async function POST(req: Request) {
  try {
    const body = await req.json();
    const parsed = BodySchema.parse(body);

    // Mock auth logic (replace with real auth)
    if (parsed.email === "demo@blogify.test" && parsed.password === "password123") {
      // Example: set cookie or return token
      return NextResponse.json({ ok: true, user: { email: parsed.email, name: "Demo User" } }, { status: 200 });
    }

    // For demo allow any email/password but show a success for convenience:
    // return NextResponse.json({ ok: true, user: { email: parsed.email } }, { status: 200 });

    // Otherwise fail:
    return NextResponse.json({ ok: false, message: "Invalid credentials (try demo@blogify.test / password123)" }, { status: 401 });
  } catch (err: any) {
    return NextResponse.json({ ok: false, message: err?.message || "Invalid request" }, { status: 400 });
  }
}

