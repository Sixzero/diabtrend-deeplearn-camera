# diabtrend-dl-camera

Camera based food recognition on the edge by DiabTrend.
The recognition is running with 30-60FPS even on low end devices and providing continuous recognitions. We differentiate between 1300 different food categories, each category has many different foods in the category.  

## Installation

```sh
npm install diabtrend-dl-camera
```

## Usage

```js
import { DeepLearnCamera } from 'diabtrend-deeplearn-camera';
import { useWindowDimensions } from 'react-native';

// ...

const apiKey = "GENERATED_API_KEY"
const RecognitionCamera = (props) => {
  const {width} = useWindowDimensions();

  return (
    <DeepLearnCamera
      style={{
        height: width,
        width: width,
      }}
      onClassification={(array_of_classifications, img_uri) => {}}
      // array_of_classifications = [{label: ID, amount: 140, confidence: 0.74}, {...}, ...]
      apiKey={apiKey}
    />
  );
}
```

- apiKey: Will be used for authorization when downloading the food recognition model. It will happen on the first instantiation of the camera. Also we use this API key for telemetry data. 
The pricing for the API key is discussed with DiabTrend. It is offered on a usage-based(only successful ones) and yearly licensing model. Contact DiabTrend to learn more: https://diabtrend.com/contact.


## Interacting with the food database

```js
export const getFoodDbRest = async ({foodDatabaseLanguage, uid}) => {
  return await fetch(foodtrend2Url + '/GET_FoodDb', {
    method: 'post',
    body: JSON.stringify({foodDatabaseLanguage, uid}),
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`,
    },
  })
    .then(res => res.json())
    .catch(console.warn);
};
```

It will return an object of `{FOOD_ID: FOOD_OBJ, ...}` mapping. Food object contains a huge amount of data about the food, nutritional informations, units of measurement and other things. It will be described in detail later on.

- foodDatabaseLanguage: The selected language of the database: Options:
`['en_GB', 'hu_HU', 'de_DE', 'it_IT', 'fr_FR', 'ro_RO', 'no_NB', 'nl_NL', 'pl_PL', 'pt_PT', 'es_ES', 'sv_SE', 'tr_TR',]`


- uid: Each user can have created foods. 

- foodtrend2Url: The url of the food database cache layer.

- apiKey: will be used to validate usage authority.


Probably it is worth caching the food database on the user's mobile, since it is around 5-10MB and is increasing in size. 

## Requesting the updates of database

Since the full download of database is costly, we should only update the cached with the new changes. 

```const newCachedFoodDb = {...cachedFoodDb, ...changes}```

The `changes` can be requested from a unix timestamp `m` or you can query a specific food if missing by specifying the `foodid`. (It can happen in some rare case, when the a specific food from the database got deleted and a user still would like to look up its data).

```js
export const getFoodDBFromM = async ({foodDatabaseLanguage, uid, m, foodid}) => {
  return await fetch(foodtrend2Url + '/GET_FoodDb_from_M', {
    method: 'post',
    body: JSON.stringify({foodDatabaseLanguage, uid, m, foodid}),
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`,
    },
  })
    .then(res => res.json())
    .catch(console.warn);
};
```

It will return an object of changes in the same format as before: `{foodid: food_obj, ...}`

One would usually calculate the `m` value from the `cachedFoodDb` by finding highest modification time stored for the foods, which can be done with:
```js
let mMax = 0;
cachedFoodDb && Object.values(cachedFoodDb).map(f => {
  if (f.m > mMax) {
    mMax = f.m;
  }
});
const {changes} = await getFoodDBFromM({foodDatabaseLanguage, uid, m: mMax, foodid});
```

- `m`: must be always specified, even if you just want to receive the data for a specific foodid, and we will always receive the new changes beside the specific `foodid`.

## Customs food databases

Custom databases can be used but needs further work. There will be a need for a mapping between the food IDs between the current food database to the custom one. 

## License

MIT

---

