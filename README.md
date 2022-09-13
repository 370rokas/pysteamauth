# Asynchronous python library for Steam authorization using protobuf

[![Code size: pysteamauth](https://img.shields.io/github/languages/code-size/sometastycake/pysteamauth)]()
[![Imports: isort](https://img.shields.io/badge/imports-isort-success)](https://pycqa.github.io/isort/)
[![Linter: flake8](https://img.shields.io/badge/linter-flake8-success)](https://github.com/PyCQA/flake8)
[![Mypy: checked](https://img.shields.io/badge/mypy-checked-success)](https://github.com/python/mypy)


## Usage

```python
import asyncio

from pysteamauth.auth import Steam


async def main():
    steam = Steam('login', 'password')
    
    await steam.login_to_steam()

    result: bool = await steam.is_authorized()

    await steam.request('https://steamcommunity.com')
    await steam.request('https://store.steampowered.com')
    await steam.request('https://help.steampowered.com')


if __name__ == '__main__':
    asyncio.run(main())
```

## If account have Steam Guard

```python
from pysteamauth.auth import Steam, AuthenticatorData

steam = Steam(
    login='login',
    password='password',
    authenticator=AuthenticatorData(
        shared_secret='shared_secret'
    )
)
```

## Cookie storage

Library uses default cookie storage `BaseCookieStorage`, which stores Steam cookies in application memory.
But you can write own cookie storage. For example, redis storage:

```python
import asyncio
import json
from typing import Dict

from aioredis import Redis
from pysteamauth.abstract import CookieStorageAbstract
from pysteamauth.auth import Steam


class RedisStorage(CookieStorageAbstract):

    redis = Redis()

    async def set(self, login: str, cookies: Dict) -> None:
        await self.redis.set(login, json.dumps(cookies))

    async def get(self, login: str, domain: str = 'steamcommunity.com') -> Dict:
        cookies = await self.redis.get(login)
        if not cookies:
            return {}
        return json.loads(cookies).get(domain, {})
    
    async def clear(self, login: str):
        await self.redis.delete(*[login])


async def main():
    steam = Steam(
        login='login',
        password='password',
        cookie_storage=RedisStorage,
    )
    
    await steam.login_to_steam()


if __name__ == '__main__':
    asyncio.run(main())

```

## Error processing

```python
import asyncio

from pysteamauth.auth import Steam
from pysteamauth.errors import SteamError


async def main():
    try:
        await Steam('login', 'password').login_to_steam()
    except SteamError as error:
        print(error)


if __name__ == '__main__':
    asyncio.run(main())
```

#### Or

```python
import asyncio

from pysteamauth.auth import Steam
from pysteamauth.errors import SteamError, custom_error_exception


class LoginError(SteamError):
    ...


class RateLimitExceeded(SteamError):
    ...


# Error codes are stored in pysteamauth.errors
custom_error_exception({
    5: LoginError,
    84: RateLimitExceeded,
})


async def main():
    try:
        await Steam('login', 'password').login_to_steam()
    except LoginError as error:
        print(error)


if __name__ == '__main__':
    asyncio.run(main())
```

#### Output
`{'error': 'InvalidPassword', 'code': 5}`

## Proto files

- https://github.com/SteamDatabase/Protobufs/blob/master/steam/enums.proto
- https://github.com/SteamDatabase/Protobufs/blob/master/steam/steammessages_base.proto
- https://github.com/SteamDatabase/Protobufs/blob/master/steam/steammessages_auth.steamclient.proto
- https://github.com/SteamDatabase/Protobufs/blob/master/steam/steammessages_unified_base.steamclient.proto
