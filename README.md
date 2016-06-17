# Margaret Fetcher

[![Build Status](https://travis-ci.org/madewithlove/margaret-fetcher.svg?branch=master)](https://travis-ci.org/madewithlove/margaret-fetcher)

Dead simple request classes for fetch.

## Usage

To use it simply create a class that extends `JsonRequest` (or `AbstractRequest`) and define requests as methods on it:

```js
import {JsonRequest} from 'margaret-fetcher';

class UserRequests extends JsonRequest
{
    query = {
        include: 'articles',
    };

    show(user) {
        return this.get(`users/${user}`);
    }

    update(user, attributes) {
        return this.put(`users/${user}`, attributes);
    }
}

export default UserRequests;
```

The `AbstractRequest` class comes with a method per HTTP verb (`this.get`, `this.post` etc.).
To then use those methods simply call the class anywhere:

```js
import UserRequests from './UserRequests';

const user = new UserRequests().show(3);
```

You can also use the `CrudRequest` class which already comes with methods for CRUD endpoints:

```js
import {CrudRequest} from 'margaret-fetcher';

class UserRequests extends CrudRequest
{
    query = {include: 'articles'};

    resource = 'users';
}

export default UserRequests;
```

```js
import UserRequests from './UserRequests';

const requests = new UserRequests;

const user = requests.show(3);

requests.update(3, {foo: 'bar'});
requests.delete(3);
```

## Advanced usage

### Configuring options

You can configure options passed with all requests either as one time thing:

```js
// Merge options with the defaults
UserRequests.withOptions({headers: {Authorization: 'Bearer FOOBAR'}}).show(3)

// Override default options
UserRequests.setOptions({headers: {Authorization: 'Bearer FOOBAR'}}).show(3)
```

Or through the class itself:

```js
class UserRequests extends CrudRequest {
    constructor() {
        super();

        this.withOptions({
            headers: {
                Authorization: `Bearer ${access_token}`,
            }
        });
    }
}
```

You can also pass callables as any option, and it will only get resolved before each request.
Useful if you need to pass options that need to be always up to date:

```js
UserRequests.withOptions({
    headers: {
        Authorization: () => `Bearer ${AuthManager.getToken()}`,
    }
})
```

### Configuring query parameters

You can configure query parameters with the same ease through these provided methods:

```js
// Override all query parameters
UserRequests.setQueryParameters({foo: 'bar'});

// Append new query parameters
UserRequests
    .withQueryParameter('foo', 'bar')
    .withQueryParameter('baz', 'qux');

UserRequests.withQueryParameters({foo: 'bar', baz: 'qux'});
```

You can also pass arrays to these methods:

```js
UserRequests.withQueryParameter('foo', ['bar', 'baz']); // ?foo[]=bar&foo[]=baz
```

### Configuring middlewares

The promise returned by `fetch` will be passed through a list of `middlewares`. By default it will return an object of the data contained in the response. But you can add your own middlewares to perform specific logic.

```js
import {parseJson} from 'margaret-fetcher';

class MyRequest extends CrudRequest {
    constructor() {
        super();

        this.setMiddlewares([
          parseJson,
          ::this.extractAuthorizationHeader,
        ]);
    }

    extractAuthorizationHeader(response) {
        const authorizationHeader = response.headers.get('Authorization');

        // Store it somewhere.

        return response;
    }
}
```

You can disable all middlewares for a given request using the `withoutMiddlewares` method:

```js
const users = new UserRequests()
    .withoutMiddlewares()
    .show(3);
```

### Extra helpers

The package also comes with some helper methods for common options:

```js
UserRequests.withBearerToken('FOOBAR').show(3)
```

You can use a function as well, like for other options:

```js
UserRequests.withBearerToken(::AuthManager.getToken).show(3)
```

## Testing

```bash
$ npm test
$ npm test:watch
```
