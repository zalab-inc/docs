# Next.js Form Handling with Safe Actions and React Hook Form

This project demonstrates a modern, type-safe approach to form handling in Next.js applications using next-safe-action, React Hook Form, and Prisma.

## Features

- 🔒 Type-safe form handling with Zod validation
- 🚀 Server actions with error handling
- 📝 React Hook Form integration
- 🎯 Prisma ORM for database operations
- ⚡ Real-time form validation
- 🔄 Automatic cache revalidation
- 🎨 Shadcn/ui components

## Tech Stack

- Next.js (App Router)
- TypeScript
- Prisma
- next-safe-action
- React Hook Form
- Zod
- shadcn/ui

## Examples

### 1. Basic Course Form

A simple form for adding courses with validation:

```typescript
// Schema (course-schemas.ts)
import { z } from "zod";

export const CreateCourseSchema = z.object({
  course: z.string().min(1, { message: "Course is required" }),
});

export type CreateCourse = z.infer<typeof CreateCourseSchema>;

// Server Action (course-actions.ts)
"use server";

import { returnValidationErrors } from "next-safe-action";
import { actionClient } from "@/lib/safe-action";
import { prisma } from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export const addCourseAction = actionClient
  .schema(CreateCourseSchema)
  .action(async ({ parsedInput }) => {
    const newCourse = await prisma.course.create({
      data: {
        course: parsedInput.course,
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    });

    if (!newCourse?.id) {
      return returnValidationErrors(CreateCourseSchema, {
        _errors: ["Invalid course data"],
      });
    }
    revalidatePath("/panel/course/listing");
    return {
      message: "Course created successfully",
      course: newCourse,
    };
  });

// Form Component (AddCourseForm.tsx)
"use client";

import { Button } from "@/components/ui/button";
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { zodResolver } from "@hookform/resolvers/zod";
import { useHookFormAction } from "@next-safe-action/adapter-react-hook-form/hooks";
import { toast } from "sonner";

export function AddCourseForm() {
  const { form, handleSubmitWithAction } = useHookFormAction(
    addCourseAction,
    zodResolver(CreateCourseSchema),
    {
      actionProps: {
        onSuccess: ({ data }) => {
          toast.success(data?.message);
        },
        onError: ({ error }) => {
          toast.error(error?.serverError);
        },
      },
      formProps: {
        defaultValues: {
          course: "",
        },
      },
      errorMapProps: {
        joinBy: " and ",
      },
    }
  );

  return (
    <Card className="max-w-md w-full mx-auto">
      <CardHeader>
        <CardTitle>Add Course</CardTitle>
        <CardDescription>Enter the details of the new course.</CardDescription>
      </CardHeader>
      <CardContent>
        <Form {...form}>
          <form onSubmit={handleSubmitWithAction} className="space-y-8">
            <FormField
              control={form.control}
              name="course"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Course</FormLabel>
                  <FormControl>
                    <Input placeholder="Enter course name" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <Button type="submit" disabled={form.formState.isSubmitting}>
              {form.formState.isSubmitting ? "Loading..." : "Add Course"}
            </Button>
          </form>
        </Form>
      </CardContent>
    </Card>
  );
}
```

### 2. Login Form Example

A more complex form with multiple fields and password handling:

```typescript
// Login Form Component
export function LoginForm() {
  const { form, handleSubmitWithAction } = useHookFormAction(
    loginAction,
    zodResolver(LoginSchema),
    {
      actionProps: {
        onSuccess: ({ data }) => {
          toast.success(data?.message);
        },
        onError: ({ error }) => {
          toast.error(error?.serverError);
        },
      },
      formProps: {
        defaultValues: {
          email: "",
          password: "",
        },
      },
    }
  );

  return (
    <Card className="mx-auto max-w-sm">
      <CardHeader>
        <CardTitle>Login</CardTitle>
        <CardDescription>
          Enter your email and password to login to your account.
        </CardDescription>
      </CardHeader>
      <CardContent>
        <Form {...form}>
          <form onSubmit={handleSubmitWithAction} className="space-y-8">
            <FormField
              control={form.control}
              name="email"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Email</FormLabel>
                  <FormControl>
                    <Input
                      type="email"
                      placeholder="johndoe@mail.com"
                      {...field}
                    />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <FormField
              control={form.control}
              name="password"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Password</FormLabel>
                  <FormControl>
                    <PasswordInput placeholder="******" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <Button type="submit" disabled={form.formState.isSubmitting}>
              {form.formState.isSubmitting ? "Loading..." : "Login"}
            </Button>
          </form>
        </Form>
      </CardContent>
    </Card>
  );
}
```

## Key Features Explained

### Safe Actions
- Type-safe server actions with automatic input validation
- Custom error handling and logging
- Secure error masking in production

### Form Integration
- Seamless integration with React Hook Form
- Real-time validation with Zod
- Loading states and error handling
- Toast notifications for success/error feedback

### Database Operations
- Prisma client with global instance management
- Automatic cache revalidation after mutations
- Type-safe database operations

## Best Practices

1. Always define schemas for form validation
2. Use server actions for data mutations
3. Implement proper error handling
4. Leverage TypeScript for type safety
5. Follow the Next.js App Router patterns
6. Use proper form state management
7. Implement loading states for better UX

## Error Handling

The system includes multiple layers of error handling:
- Client-side form validation
- Server-side input validation
- Database operation error handling
- Custom error messages with proper masking
- Toast notifications for user feedback

## Contributing

Feel free to contribute to this project by:
1. Creating issues for bugs or feature requests
2. Submitting pull requests with improvements
3. Adding more documentation or examples
4. Sharing your use cases and feedback
