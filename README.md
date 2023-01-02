# @react-libraries/next-apollo-server

Package for calling ApolloServer4 from Next.js.

'scalar Upload' is available for multipart data.
The following headers need to be added to the client-side request in order for the upload to take place.

```ts
{'apollo-require-preflight': 'true'}
```

## Sample

- Source
  <https://github.com/ReactLibraries/next-apollo-server>
- App
  - <https://github.com/SoraKumo001/next-urql>
  - <https://github.com/SoraKumo001/next-apollo-server>

### src/pages/api/graphql.ts

```ts
import { promises as fs } from "fs";
import { ApolloServer } from "@apollo/server";
import {
  executeHTTPGraphQLRequest,
  FormidableFile,
} from "@react-libraries/next-apollo-server";
import type { IResolvers } from "@graphql-tools/utils";
import type { NextApiHandler, NextApiRequest, NextApiResponse } from "next";

/**
 * Type settings for GraphQL
 */
const typeDefs = `
  # Return date
  scalar Date
  type Query {
    date: Date!
  }

  # Return file information
  type File {
    name: String!
    type: String!
    value: String!
  }
  scalar Upload
  type Mutation {
    upload(file: Upload!): File!
  }
`;

/**
 * Set Context type
 */
type Context = { req: NextApiRequest; res: NextApiResponse };

/**
 * Resolver for GraphQL
 */
const resolvers: IResolvers<Context> = {
  Query: {
    date: async (_context, _args) => new Date(),
  },
  Mutation: {
    upload: async (_context, { file }: { file: FormidableFile }) => {
      return {
        name: file.originalFilename,
        type: file.mimetype,
        value: await fs.readFile(file.filepath, { encoding: "utf8" }),
      };
    },
  },
};

/**
 * apolloServer
 */
const apolloServer = new ApolloServer<Context>({
  typeDefs,
  resolvers,
});
apolloServer.start();

/**
 * APIRoute handler for Next.js
 */
const handler: NextApiHandler = async (req, res) => {
  //Convert NextApiRequest to body format for GraphQL (multipart/form-data support).
  return executeHTTPGraphQLRequest({
    req,
    res,
    apolloServer,
    context: async () => ({ req, res }),
    options: {
      //Maximum upload file size set at 10 MB
      maxFileSize: 10 * 1024 * 1024,
    },
  });
};

export default handler;

export const config = {
  api: {
    bodyParser: false,
  },
};
```
