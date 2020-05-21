---
layout: post
title:  "Axios interceptor to convert ISO 8601 to Luxon datetime \ momentJs"
description: Axios interceptor to convert ISO 8601 to Luxon datetime \ momentJs
date:   2019-05-21 17:03:36 +0200
categories: Javascript Interceptor Luxon ISO8601
---
I got tired of converting ISO 8601 dates as string when calling REST Endpoints with axios in some postprocessing of the response. Currently this is best solution that comes to my mind, but I have lot to learn about javascript, .. so please no hates:)

Idea is to register a converter or interceptor to axios which recursivelly traverses response and replaces ISO 8601 strings to Luxon DateTime objects(or momentJS..).
For performance reasons on large responses, it might be better to use keywords approach for matching the field rather than regex.

isoConverter.js
```javascript
import { DateTime } from 'luxon';

const keywords = ['Time', 'From', 'To'];

export default {
  adaptIsoStrings(input) {
    const adaptRecursive = (obj) => {
      Object.keys(obj).forEach((key) => {
        if (obj[key] !== null && typeof obj[key] === 'object') {
          adaptRecursive(obj[key]);
        } else if (keywords.some((keyword) => key.includes(keyword))
          || /\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}.\d*Z/.test(obj[key])) {
          obj[key] = DateTime.fromISO(obj[key]);
        }
      });
    };
    adaptRecursive(input);
  },
};
```

This is how you can register interceptor.
```javascript
import axios from 'axios';
import isoConverter from './isoConverter';

axios.interceptors.response.use((response) => {
  isoConverter.adaptIsoStrings(response.data);
  return response;
});

export default {
  async getAds() {
    return Object.freeze(axios.get(`${process.env.VUE_APP_REST_API_URL}/ads`)
      .then((response) => response.data._embedded.ads)
      .catch((error) => error.response.data.message));
  },
};
```

Some test in Jest to check if its working.
```javascript
import { DateTime } from 'luxon';
import isoConverter from '@/services/isoConverter';

describe('isoConverter', () => {
  it('should adapt object by changing iso string to moments', () => {
    const dateAsIsoString = '2020-05-21T10:46:49.375803Z';
    const expectedMoment = DateTime.fromISO(dateAsIsoString);

    const objectToAdapt = {};
    objectToAdapt.nestedObject1 = {
      isoAsNull: null,
      isoAsString1Time: dateAsIsoString,
      isoAsString2: dateAsIsoString,
    };
    objectToAdapt.nestedObject2 = {
      isoAsString1From: dateAsIsoString,
      nestedObjectOfNestedObject2: {
        isoAsString2To: dateAsIsoString,
      },
    };

    isoConverter.adaptIsoStrings(objectToAdapt);
    expect(objectToAdapt.nestedObject1.isoAsString1Time).toEqual(expectedMoment);
    expect(objectToAdapt.nestedObject1.isoAsString2).toEqual(expectedMoment);
    expect(objectToAdapt.nestedObject2.isoAsString1From).toEqual(expectedMoment);
    expect(objectToAdapt.nestedObject2.nestedObjectOfNestedObject2.isoAsString2To).toEqual(
      expectedMoment,
    );
  });
});

```
