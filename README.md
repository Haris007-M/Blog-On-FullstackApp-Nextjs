# Blog-On-FullstackApp-Nextjs
It includes nextjs, nextauth, prisma , supabase, full stack app blog 


In this blog, we will be discussing how to create a full-stack application using Next.js, Prisma, Supabase, and NextAuth.

Firstly, let's discuss what each of these technologies does:

Next.js: Next.js is a React framework that provides server-side rendering and other helpful features out-of-the-box.

Prisma: Prisma is an ORM (Object-Relational Mapping) tool that makes it easy to communicate with databases using a type-safe API.

Supabase: Supabase is a cloud-based service that provides a database, authentication, and storage APIs, all in one package.

NextAuth: NextAuth is a library that simplifies authentication in Next.js apps by providing a simple API for working with various authentication providers.

Now that we know what each technology does, let's dive into creating a full-stack application using them.

To get started, we will need to create a new Next.js app. You can do this by running the following command in your terminal:
____________________________________________________________________________________________________________________________________________
lua
Copy code
npx create-next-app my-app
Next, we need to install the required dependencies:
____________________________________________________________________________________________________________________________________________
python
Copy code
npm install prisma supabase next-auth
After installing the dependencies, let's set up the Supabase client in our app. We can do this by creating a new file called supabase.js in the root directory of our app:

____________________________________________________________________________________________________________________________________________
js
Copy code
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY

____________________________________________________________________________________________________________________________________________
export const supabase = createClient(supabaseUrl, supabaseKey)
We also need to create a .env.local file in the root directory of our app and add the following:

makefile
Copy code
NEXT_PUBLIC_SUPABASE_URL=<your-supabase-url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
Now that we have our Supabase client set up, let's create a Prisma schema. We can do this by creating a new file called schema.prisma in the root directory of our app:

  ____________________________________________________________________________________________________________________________________________
prisma
Copy code
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = ["native", "rhel-openssl-1.0.x"]
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
}
  ____________________________________________________________________________________________________________________________________________
This schema defines a User model with an id, email, and password.

Next, we need to create a migration to create the User table in our database. We can do this by running the following command in our terminal:

css
Copy code
npx prisma migrate dev --preview-feature
Now that we have our database set up, let's create a new API route in Next.js that will handle authentication. We can do this by creating a new file called api/auth/[...nextauth].js:

  ____________________________________________________________________________________________________________________________________________
js
Copy code
import NextAuth from 'next-auth'
import Providers from 'next-auth/providers'
import { supabase } from '../../../supabase'

export default NextAuth({
  providers: [
    Providers.Email({
      server: process.env.EMAIL_SERVER,
      from: process.env.EMAIL_FROM,
    }),
  ],
  callbacks: {
    async signIn(user, account, profile) {
      const { email } = user
  const { error, data } = await supabase
    .from('users')
    .select('id, email, password')
    .eq('email', email)
    .single()

  if (error) {
    throw new Error(error.message)
  }

  if (!data) {
    throw new Error('No user found')
  }

  const { id, password: hashedPassword } = data

  if (hashedPassword !== user.password) {
    throw new Error('Incorrect password')
  }

  return Promise.resolve(true)
},
async jwt(token, user) {
  if (user) {
    token.id = user.id
  }

  return Promise.resolve(token)
},
async session(session, token) {
  session.user = { id: token.id }
  return Promise.resolve(session)
},
},
})
  ____________________________________________________________________________________________________________________________________________

vbnet
Copy code

This code sets up email authentication with NextAuth and uses Supabase to check if the user exists in the database and if the password is correct.

Now that we have our authentication set up, let's create a protected API route that requires authentication to access. We can do this by creating a new file called `api/user.js`:

  ____________________________________________________________________________________________________________________________________________
```js
import { getSession } from 'next-auth/client'
import { supabase } from '../../supabase'

export default async function handler(req, res) {
  const session = await getSession({ req })

  if (!session) {
    return res.status(401).json({ message: 'Unauthorized' })
  }

  const { user } = session

  const { error, data } = await supabase
    .from('users')
    .select('id, email')
    .eq('id', user.id)
    .single()

  if (error) {
    return res.status(500).json({ message: error.message })
  }

  if (!data) {
    return res.status(404).json({ message: 'User not found' })
  }

  return res.status(200).json(data)
}
  ____________________________________________________________________________________________________________________________________________
This code checks if the user is authenticated and then uses Supabase to fetch the user's data from the database.

Finally, let's create a simple UI to test our authentication and protected route. We can do this by creating a new file called pages/index.js:

  ____________________________________________________________________________________________________________________________________________
jsx
Copy code
import { useSession } from 'next-auth/client'

export default function Home() {
  const [session, loading] = useSession()

  if (loading) {
    return <p>Loading...</p>
  }F

  if (!session) {
    return (
      <>
        <p>You are not logged in.</p>
        <a href="/api/auth/signin">Log in</a>
      </>
    )
  }

  return (
    <>
      <p>You are logged in as {session.user.email}</p>
      <button onClick={async () => {
        const response = await fetch('/api/user')
        const data = await response.json()
        console.log(data)
      }}>Fetch User Data</button>
      <a href="/api/auth/signout">Log out</a>
    </>
  )
}
  ____________________________________________________________________________________________________________________________________________
This code uses the useSession hook from NextAuth to check if the user is logged in and displays a message accordingly. If the user is logged in, it displays a button to fetch the user's data from the protected route we created earlier.

And that's it! We have successfully created a full-stack application using Next.js, Prisma, Supabase, and NextAuth. Of course, this is just a starting point and there are many more things you can do with these technologies.

For example, you can expand on the user data fetched from the protected route and display more information in the UI. You can also add more authentication providers to NextAuth, such as Google or Facebook, and customize the login and signup flows.

Furthermore, you can use Prisma to define and manage your database schema and migrate changes to it. Prisma also supports many popular databases, such as PostgreSQL, MySQL, and SQLite, so you can easily switch between them if needed.

Supabase also provides other features besides authentication, such as database management, real-time subscriptions, and storage. You can explore these features in the Supabase documentation and see how they can be integrated with your application.

Overall, Next.js, Prisma, Supabase, and NextAuth provide a powerful and flexible stack for building modern web applications. They offer a great balance of productivity and control, allowing you to focus on building your application's features while also ensuring security and scalability.

I hope this tutorial has been helpful and has given you some ideas for your next project. Happy coding!



Before we wrap up, let's summarize what we've learned in this tutorial:

Next.js is a powerful and flexible framework for building server-rendered React applications.
Prisma is a modern database toolkit that simplifies database access and management.
Supabase is an open-source Firebase alternative that provides a suite of backend services, such as authentication, database management, and storage.
NextAuth is a library for adding authentication to Next.js applications, supporting various providers and customization options.
Together, these technologies provide a powerful and flexible stack for building modern web applications.
In this tutorial, we've created a simple full-stack application that uses Next.js for the frontend, Prisma for the database access and management, Supabase for the authentication and database hosting, and NextAuth for the authentication flow. We've seen how easy it is to set up and integrate these technologies, and how they can work together to provide a seamless and secure user experience.

If you're interested in learning more about these technologies, I encourage you to check out their respective documentation and explore their capabilities further. There's always more to learn and discover, and with these tools at your disposal, you can build amazing web applications that are both powerful and easy to manage.

Thank you for following along with this tutorial, and happy coding!



As a final step, you may want to deploy your application to a production environment. Next.js supports various deployment options, such as Vercel, AWS, and Google Cloud, which can automatically build and deploy your application to the cloud.

You can also use Supabase's hosting service to deploy your frontend and backend to the same domain, making it easier to manage and secure your application.

To do this, you can run the following command to build your Next.js application:

Copy code
npm run build
This will generate an optimized production build of your application in the ./out directory. You can then deploy this directory to a hosting provider of your choice.

For example, if you want to deploy your application to Vercel, you can create an account and connect your GitHub repository. Vercel will automatically detect your Next.js application and build and deploy it to a unique URL.

You can also configure your deployment settings in Vercel to use environment variables for sensitive information, such as API keys and database credentials.

Once your application is deployed, you can test it and make sure everything works as expected. You may also want to monitor your application's performance and security, and optimize it for better user experience.

In conclusion, building a full-stack application with Next.js, Prisma, Supabase, and NextAuth is a powerful and flexible approach to modern web development. It allows you to focus on your application's features and functionality, while also ensuring security and scalability.

I hope this tutorial has been helpful and has inspired you to explore these technologies further. Happy coding!
