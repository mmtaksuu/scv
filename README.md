
Simple C Vector
===============

Copyright 2003-2014 Joergen Ibsen

<http://www.ibsensoftware.com/>

[![Build Status](https://travis-ci.org/jibsen/scv.png?branch=master)](https://travis-ci.org/jibsen/scv) [![codecov.io](http://codecov.io/github/jibsen/scv/coverage.svg?branch=master)](http://codecov.io/github/jibsen/scv?branch=master)


About
-----

Simple C Vector (scv) is an ANSI C implementation of a [dynamic array][dyna],
with an interface similar to C++ [std::vector][vector].

I wrote it for a project years ago, and decided to dust it off and make it
available under the [Apache License](LICENSE).

[dyna]: http://en.wikipedia.org/wiki/Dynamic_array
[vector]: http://en.cppreference.com/w/cpp/container/vector


Usage
-----

The include file `scv.h` contains documentation in the form of [doxygen][]
comments. A configuration file is included, so you can simply run `doxygen`
to generate documentation in HTML format.

scv uses [CMake][] to generate build systems. To create one for the tools on
your platform, and build scv, use something along the lines of:

~~~sh
mkdir build
cd build
cmake ..
cmake --build .
~~~

You can also simply compile the source files and link them into your project.
CMake just provides an easy way to build and test across various platforms and
toolsets.

[doxygen]: http://www.doxygen.org/
[CMake]: http://www.cmake.org/


Example
-------

Here is an example that reads integer coordinates in the form `x,y`
into a `scv_vector` and prints them in lexicographical order:

~~~c
#include <stdlib.h>
#include <stdio.h>

#include "scv.h"

struct point {
	int x;
	int y;
};

int point_compare(const struct point *lhs, const struct point *rhs)
{
	if (lhs->x == rhs->x) {
		return lhs->y - rhs->y;
	}

	return lhs->x - rhs->x;
}

int main(void)
{
	struct scv_vector *v;
	struct point p;
	size_t i;

	/* create a scv_vector of points, reserving space for 10 */
	v = scv_new(sizeof p, 10);

	/* read coordinates into p and append to v */
	while (scanf("%d,%d", &p.x, &p.y) == 2) {
		scv_push_back(v, &p);
	}

	/* sort points in v */
	qsort(scv_data(v), scv_size(v), scv_objsize(v), point_compare);

	/* print points */
	for (i = 0; i < scv_size(v); ++i) {
		struct point *pp = scv_at(v, i);

		printf("%d,%d\n", pp->x, pp->y);
	}

	scv_delete(v);

	return 0;
}
~~~


Details
-------

The user is responsible for passing pointers to valid `scv_vector` structures.
The code contains assertions to help while debugging.

To enable the code to work in general, independent of the type of objects
stored in a particular `scv_vector`, functions for inserting and accessing
elements take and return `void` pointers.

This has the drawback that you cannot use a constant expression directly, but
need an actual object that you can pass a pointer to, when calling functions
like `scv_push_back()`.

Pointers into the memory used to hold the elements of a `scv_vector`, will be
valid until a function causes a reallocation. If you know the element type,
and speed is important, you can use pointers to access the elements directly:

~~~c
	/* v is a scv_vector of int, get a pointer to it's data */
	int *p = scv_data(v);

	/* we can access the elements of v using p */
	p[5] = 42;
~~~

scv uses `malloc()` for memory allocation, which works well in many cases,
but if needed, you could improve performance by using a custom allocator.
