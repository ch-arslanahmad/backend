# Separation of Concerns

Separation of concerns is a software design principle used when building, designing applications, systems, etc.

It states that each part of a program is responsible for only one concern or responsibility.

It allows modularity, and separates responsibilities, concerns, and features by organizing them within a directory, file, module, or function.

> [!note]
> A module is a piece of reusable code that performs one type of job.
> It is unlike a **function** which performs only an operation.

It is closely related to the:

- Single Responsibility Principle (SRP)
- Modularity
- Encapsulation
- Layered Architecture (Controller → Service → Repository)
- MVC

Because everything is organized and handled in some kind of hierarchy, it becomes:
- Easier to understand, maintain, test, and reuse.
- Easier for multiple people to work independently, because each problem, concern, or feature is isolated.


## Example

For example, we are creating an app for:
- recruiters to post jobs, reject candidates, and shortlist candidates,
- candidates to apply for jobs.

Here we can separate the concerns into different modules, for example:
- `recruiter` module,
- `candidate` module.

Or we could divide it into different modules based on the features, for example: 

```
├── backend
│   ├── seed.ts
│   ├── src
│   │   ├── classes/        # User.ts, Job.ts, Application.ts
│   │   ├── db/             # database.ts, schema.sql
│   │   ├── middleware/     # auth.ts
│   │   ├── routes/         # auth, jobs, applications, savedJobs
│   │   ├── server.ts
│   │   ├── types/          # express.ts
│   │   └── utils/          # pdf.ts
│   └── tsconfig.json
├── frontend
│   ├── src
│   │   ├── api.ts
│   │   ├── components/     # NavBar, JobCard, JobBoard, AuthModel
│   │   ├── main.tsx
│   │   ├── pages/          # Home, Login, Dashboard, JobListing, JobDetail, Application, ErrorPage
│   │   └── types.ts
│   └── vite.config.js
├── package.json
└── README.md
```

Now inside the module we can further separate the concerns into different files, for example:
- Inside `src/classes`, each entity (`User.ts`, `Job.ts`, `Application.ts`) has its own file.
- Inside `src/routes`, each resource (`auth.ts`, `jobs.ts`, `applications.ts`, `savedJobs.ts`) has its own file.