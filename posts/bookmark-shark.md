
### Overview of the project

This project is a link sharing platform, using my own custom authentication, along with googles oauth platform.  I've also provided my own email validation, with help from [Lucia-Auth](https://lucia-auth.com/) It's deployed with next and uses material ui's component library, along with daisy-ui for some extra styling.  It's built 100% by me, and It's currently not ready for production, but it's close.  The code is hosted on [GitHub](https://github.com/cjodo/bookmark-shark-next/tree/main), and instructions for getting the dev server running are in the readme.

My goal for this project first was just a hobby project to get more familiar with next, but then as I got further into the project, I found there's actually a place that this platform can help people.  The current platforms that people share links, are things like Twitter, Pinterest, Reddit, etc. Although these platforms I use pretty regularly, they support so much more than sharing links.

What I think this could help people with, is finding ultra specific and related content.  To help people find the websites they're looking for with certain research topics, to movie and tv theories and info, to learning resources in any field, the list is endless. 

## Tech Stack

I chose to go with Next.js for the framework.  I like typescripts flexible type system, and the server side rendering to allow security and speed when delivering pages.  It also helps with SEO as google's indexing algorithm doesn't love Client-Side Rendering.

For Authentication and signing up, I don't like how cumbersome some of the 'batteries included' auth libraries I've tried in the past are (Clerk, and NextAuth).  The reason for this, is that it just adds this huge overhead and another layer of complexity to the app. I also really don't like code executing that I'm not in control of.  

For the database, I chose to use Prisma's ORM, it allows for the data models from the Postgres database to be represented in intuitive methods and interfaces defined in typescript. Prisma has become the defacto ORM when working with typescript and next.js applications.


## What I learned

The biggest challenge for me, was porting the example auth given by [Lucia-Auth](https://lucia-auth.com/) as mentioned above into prisma.  The examples are written in raw SQLite queries, but there are so many dfferent examples to work with, it made this pretty easy.

### For example

- Porting this function from sqlite
```ts
export async function createUser(email: string, username: string, password: string): Promise<User> {
	const passwordHash = await hashPassword(password);
	const recoveryCode = generateRandomRecoveryCode();
	const encryptedRecoveryCode = encryptString(recoveryCode);
	const row = db.queryOne(
		"INSERT INTO user (email, username, password_hash, recovery_code) VALUES (?, ?, ?, ?) RETURNING user.id",
		[email, username, passwordHash, encryptedRecoveryCode]
	);
	if (row === null) {
		throw new Error("Unexpected error");
	}
	const user: User = {
		id: row.number(0),
		username,
		email,
		emailVerified: false,
		registered2FA: false
	};
	return user;
}
```

- to prisma 
```ts
export async function createUser(email:string, username:string, password:string): Promise<Partial<User> | null> {
	const passwordHash = await hashPassword(password);
	const recoveryCode = generateRandomRecoveryCode();
	const encryptedRecoveryCode = encryptString(recoveryCode);

	console.log({recoveryCode, encryptedRecoveryCode});

	const newUser = await prisma.user.create({
		data: {
			email: email,
			username: username,
			passwordHash: passwordHash,
			recoveryCode: encryptedRecoveryCode
		},
	})

	if (newUser === null) {
		throw new Error("Unexpected error");
	}
	const user: Partial<User> = {
		id: newUser.id,
		username,
		email,
		emailVerified: false,
	};

	return user;
}
```

As you can see, the power of the orm shines here, as the User type is already defined and returned when we create a user.
- It's important to note that I returned a `Partial<User>` just to keep things safer, I didn't want any passwordHash's being sent to the client 

There are a lot of examples like this in the repo.  My goal is to release this to production in late April to early May.
