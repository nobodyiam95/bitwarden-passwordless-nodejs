# Passwordless Node.js SDK
![Build](https://github.com/bitwarden/passwordless-nodejs/actions/workflows/ci.yml/badge.svg)
[![Maven Central](https://img.shields.io/npm/v/@passwordlessdev/passwordless-nodejs.svg)](https://www.npmjs.com/package/@passwordlessdev/passwordless-nodejs)
[![Downloads](https://img.shields.io/npm/dm/@passwordlessdev/passwordless-nodejs.svg)]([https://pypi.org/project/passwordless/](https://www.npmjs.com/package/@passwordlessdev/passwordless-nodejs))

The official Bitwarden Passwordless.dev Node.js library.

## Installation
```bash
$ npm i @@passwordlessdev/passwordless-nodejs
```

## Dependencies & Requirements
- ES2018 or newer, read more [here](https://node.green/).
- Supported JavaScript modules: CommonJS, ES
- Node.js 10 or newer

## Getting Started
Follow the [Get started guide](https://docs.passwordless.dev/guide/get-started.html).

### Customization
When using `.env` to configure your web application you'll need to set the following properties:

| Key                   | Value example                                  | Description                                                  | Optional |
|-----------------------|------------------------------------------------|--------------------------------------------------------------|----------|
| `PASSWORDLESS_API`    | `https://v4.passwordless.dev`                  | The base url where your Passwordless.dev back-end is running | Yes      |
| `PASSWORDLESS_SECRET` | `demo:secret:f831e39c29e64b77aba547478a4b3ec6` | This is your secret obtained from the AdminConsole.          | No       |

You would then use the PasswordlessClient as:

### Creating a `PasswordlessClient` instance
Specifying the `baseUrl` would be optional, and will contain `https://v4.passwordless.dev` as its default value.

```TSX
const options: PasswordlessOptions = {
  baseUrl: 'https://v4.passwordless.dev'
};
this._passwordlessClient = new PasswordlessClient('demo:secret:f831e39c29e64b77aba547478a4b3ec6', options);
```

```TSX
const options: PasswordlessOptions = {};
this._passwordlessClient = new PasswordlessClient('demo:secret:f831e39c29e64b77aba547478a4b3ec6', options);
```

### Registration
If you had for example a 'UserController.ts' with a 'signup' arrow function. You could register a new token as shown below.

You'll first want to proceed to store the new user in your database and verifying it has succeeded, before registering the token.

```TSX
signup = async (request: express.Request, response: express.Response) => {
        const signupRequest: SignupRequest = request.body;
        const repository: UserRepository = new UserRepository();
        let id: string = null;
        try {
            id = repository.create(signupRequest.username, signupRequest.firstName, signupRequest.lastName);
        } catch {
            // do error handling, creating user failed.
        } finally {
            repository.close();
        }

        if (!id) {
            // Do not proceed to create a token, we failed to create a user.
            response.send(400);
        }

        let registerOptions = new RegisterOptions();
        registerOptions.userId = id;
        registerOptions.username = signupRequest.username;
        if (signupRequest.deviceName) {
            registerOptions.aliases = new Array(1);
            registerOptions.aliases[0] = signupRequest.deviceName;
        }
        
        registerOptions.discoverable = true;
        
        const token: RegisterTokenResponse = await this._passwordlessClient.createRegisterToken(registerOptions);
        response.send(token);
    }
```

### Logging in

```TSX
signin = async (request: express.Request, response: express.Response) => {
        try {
            const token: string = request.query.token as string;
            const verifiedUser: VerifiedUser = await this._passwordlessClient.verifyToken(token);

            if (verifiedUser && verifiedUser.success === true) {
                // If you want to build a JWT token for SPA that are rendered client-side, you can do this here.
                response.send(JSON.stringify(verifiedUser));
                return;
            }
        } catch (error) {
            console.error(error.message);
        }
        response.send(401);
    }
```

### Examples
See [Passwordless Node.js Example](https://github.com/bitwarden/passwordless-nodejs/blob/main/examples/simple-example) for a Node.js Web application.

## Documentation
For a comprehensive list of examples, check out the [API documentation](https://docs.passwordless.dev/guide/get-started.html).
