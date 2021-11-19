# Digdir

## Getting Started

...

### Prerequisites

* Miniconda (https://docs.conda.io/en/latest/miniconda.html)
* Test and/or production Self-Service Client (https://docs.digdir.no/maskinporten_sjolvbetjening_api.html#selvbetjening-som-api-konsument)
* Test and/or production PKCS #12 file (Buypass or Commfides)

### Installation

1. Clone the repo
```sh
git clone https://github.com/torleifg/digdir.git
```

2. Add channel
```sh
conda config --add channels conda-forge
conda config --set channel_priority strict
```

3. Create environment
```sh
conda create --name <environment> jwcrypto pyjwt urllib3
```

4. Activate environment
```sh
conda activate <environment>
```

5. Add PKCS #12 file
```sh
mkdir /cert
cp <path>/<*.p12> cert/*.p12
```

6. Modify configuration
```sh
cd config

cp selvbetjening.ini selvbetjening-local.ini
vim selvbetjening-local.ini

cp maskinporten.ini maskinporten-local.ini
vim maskinporten-local.ini
```

## Usage

### Selvbetjening

```python
import digdir, json

s = digdir.Selvbetjening('config/selvbetjening-local.ini', 'VER2')

jwt_grant = s.create_jwt_grant()
access_token = s.get_access_token(jwt_grant)

client = s.create_client(access_token, 'client_name', 'scope1 scope2')

client_id = json.loads(client.data.decode('utf-8'))['client_id']
jwks = digdir.create_jwks('kid')

f = open('keyset.jwks', 'w')
f.write(jwks.export())
f.close()

response = s.add_keyset_to_client(access_token, client_id, jwks.export())

...
```

### Maskinporten

```python
import digdir, json

m = digdir.Maskinporten('config/maskinporten-local.ini', 'VER2')

f = open('keyset.jwks')
jwks = f.read()
f.close()

jwt_grant = m.create_jwt_grant('client_id', 'scope', 'kid', jwks)
access_token = m.get_access_token(jwt_grant)

person = m.get_krr_person(access_token, 'person_id')

...
```