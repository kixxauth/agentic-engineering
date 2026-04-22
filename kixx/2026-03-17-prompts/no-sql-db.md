I need you to create a detailed specification for a custom no-sql database.

Here is the background context:

<background>
The user is the primary maintainer of an open source web development framework called Kixx. The Kixx Framework supports applications built on server-side JavaScript platforms like Node.js, Deno, and Bun.

The Kixx Framework provides primitive capabilities for web applications like HTTP routing and HTML templating. Now the user wants to add data storage capability as well.

You should note that applications built with the Kixx Framework are deployed on various target platforms including Linux virtual machines, Cloudflare Workers, and AWS Lambda functions.
</background>

Here are some user stories:

## User Story - Platform Support
As the system architect and primary maintainer of the Kixx Framework I only want to support this data store on Linux, Mac, and Windows machines which are running Node.js. For the first iteration of this datastore I do not want the added complexity of supporting other platforms. Future iterations may support additional platforms like Deno and Cloudflare Workers.

## User Story - Self Contained
As a user of the Kixx Framework for my web project I should have access to this database without installing any dependencies other than the Kixx Framework. When I have kixx installed as an npm dependency, it should already include everything I need to use this database.

## User Story - No Dependencies
As the lead contributor to the Kixx Framework I have committed to having as few dependencies as possible and insist that this database does not require any additional npm or vendored dependencies. Everything should be built from scratch, in JavaScript.

## User Story - Cross Platform
As a user of the Kixx Framework I develop my application using Node.js on a Unix based machine, but I have co-collaborators who use a mix of Windows, Mac, and Linux development machines. I expect this no-sql database to work identically across all of them.

## User Story - JSON Documents
As an application developer and user of this database I want to use JSON documents as the primitive atomic unit of storage so that I can easily update the shape of my data based on new use cases without executing fragile schema migrations.

I want to store JSON documents with a "type" attribute, like "Customer" or "Product", and I want to be able to assign a "sortKey" attribute with values like the last update time or category name. I want to be able to query all JSON documents in the database by type and return results sorted by sortKey without having to set up a specific index for it.

I expect to be able to write a JSON document to the store so long as it has an "id" and "type" attribute, and, optionally, a "sortKey" attribute.

I expect to be able to get a document using the "id" and "type".

## User Story - Document Identifiers
As the system architect for the Kixx Framework I want to mandate that every JSON document has an "id" attribute which can be hashed with a hashing algorithm. This makes desiging distributed data stores possible at larger scales.

## User Story - Query
As an application developer and user of this database I want to query on any top level attribute of a set of JSON documents. I *DO NOT* expect to query on nested attributes.

If I need to configure indexes before I can query them, that is a trade-off I can make.

## User Story - Indexes
As the system architect for the Kixx Framework I DO NOT want to offer dynamic queries or query paths into JSON documents. This would lead to additional complexity which could not be implemented by some of underlying storage engines we will eventually use.

To keep the data store simple and efficient we should require that the user configures indexes by providing a JSON document top level attribute name we intend to query on in addition to the type. In other words, an index should encapsulate a type and an attribute name.

In addition, we should provide a default index which allows users to query by the pre-configured attributes "type" and "sortKey".

## User Story - Batch Updates
As the system architect for the Kixx Framework I DO NOT want to offer users the ability to make batch updates to the database. This is a feature we may add someday, but for now we want to start with the simplest implementation.

## User Story - Conflict Resolution
As a user of the datastore I DO NOT want the datastore to handle JSON document conflict resolution internally. Instead I want to be able to create my own conflict resolution algorithms tailored to my specific use cases.

## User Story - Handle Race Conditions
As a user of the datastore I expect the datastore to handle race conditions on a single JSON document to avoid corrupting the document with partial overwrites.

## Architectural Design Constraints
The Kixx Framework allows developers to build their applications and deploy them to multiple targets, including Node.js Linux servers, Cloudflare Workers, and AWS Lambda functions, without any changes to the application code. This is accomplished through the use of the dependency inversion principle: we create concrete implementations of lower level components which are platform specific and then higher level abstractions can use these components to provide a common interface. Only the lower level components are swapped out when the deployment target changes.

**NOTE** that today we only support Node.js running on a Mac, Linux, or Windows system, but have plans to deploy on Cloudflare Workers and AWS Lambda functions.

To see an example of how the inverted dependency principle works, review the config.js module, which uses a lower level ConfigStore:

`config.js`:

```javascript
import deepMerge from '../utils/deep-merge.js';

/**
 * @typedef {Object} ConfigStore
 * @property {function(string, function(Object): void): ConfigStore} on - Registers a listener for 'update:config' or 'update:secrets'; returns this for chaining
 * @property {function(): Promise<Object>} loadConfig - Loads configuration and emits 'update:config' with the values
 * @property {function(): Promise<Object>} loadSecrets - Loads secrets and emits 'update:secrets' with the values
 */

export default class Config {

    #values = {};
    #secrets = {};

    /** Store that emits 'update:config' and 'update:secrets' when config files are loaded */
    #store = null;

    /**
     * @param {ConfigStore} store - Config store with on(eventName, listener) that emits 'update:config' and 'update:secrets'
     * @param {string} environment - Environment name for config/secrets overrides (e.g., 'development', 'production')
     */
    constructor(store, environment) {

        this.#store = store;

        this.#store.on('update:config', (values) => this.#updateConfig(values));
        this.#store.on('update:secrets', (values) => this.#updateSecrets(values));

        Object.defineProperties(this, {
            environment: {
                enumerable: true,
                value: environment,
            },
        });
    }

    getNamespace(namespace) {
        const values = this.#values[namespace];
        if (values) {
            return structuredClone(values);
        }
        return {};
    }

    getSecrets(namespace) {
        const values = this.#secrets[namespace];
        if (values) {
            return structuredClone(values);
        }
        return {};
    }

    #updateConfig(rootConfig) {
        const environmentConfig = rootConfig?.environments?.[this.environment] || {};
        this.#values = deepMerge(structuredClone(rootConfig || {}), environmentConfig);
    }

    #updateSecrets(rootSecrets) {
        const environmentSecrets = rootSecrets?.environments?.[this.environment] || {};
        this.#secrets = deepMerge(structuredClone(rootSecrets || {}), environmentSecrets);
    }
}
```

`memory-config-store.js`:

```javascript
import { EventEmitter } from 'node:events';

export default class JSModuleConfigStore {

    #emitter = new EventEmitter();
    #config = null;
    #secrets = null;

    constructor(options) {
        // Simply store configs and secrets in memory.
        this.#config = options.config;
        this.#secrets = options.secrets;
    }

    on(eventName, listener) {
        this.#emitter.on(eventName, listener);
        return this;
    }

    async loadConfig() {
        this.#emitter.emit('update:config', this.#config);
        return this.#config;
    }

    async loadSecrets() {
        this.#emitter.emit('update:secrets', this.#secrets);
        return this.#secrets;
    }
}
```

Our new no-sql database should work the same way: It should use a lower level concrete implementation of an abstract interface which deals with the file system so the higher level user interface can be deployed anywhere.

Here is a diagram of an application which can interact with different storage engines through an abstract interface:

┌───────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                                   │
│                     Application - Can switch between storage implementations                      │
│                                   without changing any code.                                      │
│                                                                                                   │
└─────────────┬──────────────────────────────────┬──────────────────────────────────┬───────────────┘
              │                                  │                                  │                
              │                                  │                                  │                
┌─────────────┴───────────────┐    ┌─────────────┴───────────────┐    ┌─────────────┴───────────────┐
│                             │    │                             │    │                             │
│  Common Abstract Interface  │    │  Common Abstract Interface  │    │  Common Abstract Interface  │
│                             │    │                             │    │                             │
└─────────────┬───────────────┘    └─────────────┬───────────────┘    └─────────────┬───────────────┘
              │                                  │                                  │                
┌─────────────┴───────────────┐    ┌─────────────┴───────────────┐    ┌─────────────┴───────────────┐
│                             │    │                             │    │                             │
│   Node.js Implementation    │    │  AWS Lambda Implementation  │    │  Cloudflare Implementation  │
│                             │    │                             │    │                             │
│                             │    │                             │    │                             │
│                             │    │                             │    │                             │
│                             │    │                             │    │                             │
└─────────────────────────────┘    └─────────────────────────────┘    └─────────────────────────────┘


Here are the technical constraints for our planned deployment targets:

**Cloudflare Workers** - Could use the [D1 Database](https://developers.cloudflare.com/d1/worker-api/d1-database/) (SQLite wrapper) and the [KV Store](https://developers.cloudflare.com/kv/) (key/value store). Any design you come up with for the abstract interface should be able to be implemented with the D1 and KV Cloudflare products for future use. DO NOT implement the Cloudflare workers solution yet. This is only noted for your awareness.

**AWS Lambda** - Could use the DynamoDB product, so any design you come up with for abstract interface should be able to be implemented with DynamoDB. DO NOT implement the AWS Lambda solution yet. This is only noted for your awareness.

**Node.js on Windows, Mac, or Linux** - Use the file system or SQLite, but do not pull in any database engine which requires a new dependency. This project uses Node.js <= 24 which has support for SQLite. The Node.js SQLite support is documented here: https://nodejs.org/docs/latest/api/sqlite.html . Note that SQLite has support for [generated columns](https://sqlite.org/gencol.html) and [json extraction](https://www.sqlite.org/json1.html), which could work well for your use cases. Of the 3 potential deployment targets the Node.js on Windows, Mac, or Linux target is the ONLY TARGET you will be implementing today. The others are for a later date.

<system>
Think carefully about possible use cases and tradeoffs. Be creative with your solutions and think outside the box. Ask the user questions if you need more clarity on the user stories, design constraints, or just want to talk through your solutions and trade-offs.
</system>
