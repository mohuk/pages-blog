---
layout: post
title: Smart way of testing Angular JS filters
date: 2014-08-16 10:22:03.000000000 +05:00
tags:
- AngularJS
status: publish
type: post
published: true
author:
  login: mohuk
  email: mohammad.umair.k@gmail.com
  display_name: mohuk
  first_name: 'Muhammad Umair'
  last_name: 'Khan'
---
Writing clean, scale-able and maintainable unit tests is as important as writing application source code with all those qualities. Pragmatic Programmer (yes, the famous book) says that developers are constantly in maintenance mode due to various reasons which means that the tests we write need to be updated constantly. One might think, why so much of overhead? While unit tests may contain tons of other advantages, to me, unit tests give me the confidence I need for refactoring.

I want to go over a small example of how one can make its unit tests better in context of testing Angular JS custom filters. Filters usually take in a bunch of arguments and return a Javascript object (primitive or list). So technically, it has few arguments and an output. Lets look at a small custom filter I made for client side pagination with UI-Bootstrap's Pagination Directive.

```javascript
angular.module("kuangular").filter('kuPagination', function () {
    return function (list, currentPage, recordsPerPage) {
        if (angular.isUndefined(list) || list.length <= 0)
            throw ("List either undefined or empty");

        if (angular.isUndefined(currentPage) || angular.isUndefined(recordsPerPage))
            throw ("Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]");

        currentPage = currentPage - 1;
        var startSelectionIndex, endSelectionIndex;
        startSelectionIndex = currentPage * recordsPerPage;
        endSelectionIndex = startSelectionIndex + recordsPerPage;

        return list.slice(startSelectionIndex, endSelectionIndex);
    };
})
```

Simple isn't it ?

Anyways, here is the test for the filter ...

```javascript
describe('Filter Pagination', function () {

    var paginationFilter, usersList;

    beforeEach(module('kuangular'));
    beforeEach(module('Mocks')); //contains the mock objects

    beforeEach(inject(function ($filter, UserManagementObjects) {
        paginationFilter = $filter('kuPagination');
        usersList = UserManagementObjects.UsersList;
    }));

    it('should throw an error if list is empty', function () {
        var filterfunc = function () {
            paginationFilter([]);
        }
        expect(filterfunc).toThrow("List either undefined or empty");
    });

    it('should throw an error if list is undefined', function () {
        var filterfunc = function () {
            paginationFilter(undefined);
        }
        expect(filterfunc).toThrow("List either undefined or empty");
    });

    it('should throw an error if list is defined but current page and/or records per page are undefined', function () {
        var filterFunc = {
            recordsPerPageUndefined: function () {
                paginationFilter(usersList, 1, undefined);
            },
            currentPageUndefined: function () {
                paginationFilter(usersList, undefined, 5);
            },
            bothUndefined: function () {
                paginationFilter(usersList, undefined, undefined);
            }
        }

        expect(filterFunc.recordsPerPageUndefined).toThrow("Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]");
        expect(filterFunc.currentPageUndefined).toThrow("Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]");
        expect(filterFunc.bothUndefined).toThrow("Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]");
    });

    it('should return a sliced list according to current page and records per page', function () {
        var retVal, testVal = [];

        retVal = paginationFilter(usersList, 1, 5);
        expect(retVal).toEqual(usersList);

        testVal = [];
        retVal = paginationFilter(usersList, 2, 2);
        testVal.push(usersList[2], usersList[3]);
        expect(retVal).toEqual(testVal);

        testVal = [];
        retVal = paginationFilter(usersList, 3, 2);
        testVal.push(usersList[4]);
        expect(retVal).toEqual(testVal);
    })
});
```

I did not feel comfortable at all with the quality of unit tests. It violates DRY principle a lot. It would require more effort to add/delete/update tests and if the API changes, it would require the developer to update all the unit tests with the updated API. To resolve the above issues I came up with an alternative way of testing the filter ...

```javascript
describe('Filter Pagination', function () {

    var paginationFilter, usersList;

    beforeEach(module('kuangular'));
    beforeEach(module('Mocks'));

    beforeEach(inject(function ($filter, UserManagementObjects) {
        paginationFilter = $filter('kuPagination');
        usersList = UserManagementObjects.UsersList;
    }));

    describe('Exception Specs', function(){
        var exceptionSpecs = [
            {
                input:[usersList, 1, undefined],
                output:"Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]"
            },
            {
                input:[usersList, undefined, 5],
                output:"Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]"
            },
            {
                input:[usersList, undefined, undefined],
                output:"Parameters for filter are not defined. [Param 1: current page, Param 2: records per page]"
            },
            {
                input:[undefined],
                output:"List either undefined or empty"
            },
            {
                input:[],
                output:"List either undefined or empty"
            },

        ]

        it('should throw an error if list is defined but current page and/or records per page are undefined', function () {
            function paginationFilterWrapper (input) {
                paginationFilter.apply(this, input);
            }
            angular.forEach(exceptionSpecs, function(spec){
                expect(paginationFilterWrapper.bind(this, spec.input)).toThrow(spec.output);
            });
        });
    });

    describe('List value specs', function(){
        var listSpecs = [
            {
                input: [usersList,1,5],
                output:usersList
            },
            {
                input: [usersList,2,2],
                output:Array.prototype.concat(usersList[2], usersList[3])
            },
            {
                input: [usersList, 3, 2],
                output:[usersList[4]]
            }
        ]

        it('should return a sliced list according to current page and records per page', function () {
            angular.forEach(listSpecs, function(spec){
                expect(paginationFilter.apply(this, spec.input)).toEqual(spec.output);
            });
        });

    });
});
```

Each describe block in the above tests is responsible for a type of unit tests, i.e. exceptions and normal tests. Each contains a collection of specs that are fed to a routine inside the "it" block (which defines the test). These  tests are more maintainable and reduce the number of "it" blocks to exactly one. Add a spec on the list to add another test, pretty darn easy. Moreover it is pretty adaptable to the API changes as well and can surely make the lives of the developers much easier.
