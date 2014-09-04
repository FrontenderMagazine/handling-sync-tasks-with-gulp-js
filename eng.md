<section class="post-content">
[Gulp JS][1] is a build system/task runner geared towards front end web
development, and is an up and coming alternative to the popular[Grunt JS][2]
system.

One of the several things that makes Gulp different from Grunt is that all of
the tasks (by default) are run asynchronously (wow, I didn't even have to look 
up the spelling and got it right on the first try!). More or less, this means 
that all tasks are run at the same time.

I recently dug in deep to Gulp, and one of the things I most struggled with was
the couple tasks I needed to run synchronously to run... synchronously.

The docs touch on it, but I was still confused with a couple things before I
finally got it.

First of all, there are 3 ways to make a task synchronous, but it has to be
combined with a**task dependency** to work.

First off, the 3 ways:

**Passing in a callback:**

    gulp.task('sync', function (cb) {  
        // setTimeout could be any async task
        setTimeout(function () {
            cb();
        }, 1000);
    });
    

**Returning a stream:**

    gulp.task('sync', function () {  
        return gulp.src('js/*.js')
            .pipe(concat('script.min.js')
            .pipe(uglify())
            .pipe(gulp.dest('../dist/js');
    });
    

**Returning a [Promise][3]:**

    gulp.task('sync', function () {  
        var deferred = Q.defer();
        // setTimeout could be any async task
        setTimeout(function () {
            deferred.resolve();
        }, 1000);
        return deferred.promise;
    });
    

Let's say we have another task that depends on the `sync` task we created above
(any one of them will do
).

It's not enough just to do the above, we have to declare our `sync` task as a
dependecy for another task:

    gulp.task('secondTask', ['sync'], function () {  
        // this task will not start until
        // the sync task is all done!
    });
    

The mistake I made, was I thought if I chained dependecies, it would wait for
earlier dependencies to finish before doing subsequent tasks.**The following
will NOT do that:**

    gulp.task('thirdTask', function () {  
        // note this task has no dependent tasks
    });
    
    // I hoped this would run sync task THEN
    // thirdTask and THEN default, but it
    // DOES NOT. It runs sync and thirdTask 
    // at the SAME TIME, then it runs default.
    gulp.task('default', ['sync', 'thirdTask'], function () {  
        // do stuff
    });
    

To make the `default` do what I originally thought it would do, I need to *also
* make `thirdTask` depend on `sync`:

    gulp.task('thirdTask', ['sync'] function () {  
        // this now depends on sync. If it returns a stream,
        // then default won't be run until thirdTask finishes
    });
    
    gulp.task('default', ['sync', 'thirdTask'], function () {  
        // do stuff
    });
    

Note what this might do if you have a `watch` task that runs `thirdTask`. Every
time you run`thirdTask`, it will first run `sync` now, and that may not be
desirable!

Anyway, hope that that can help someone else trying to figure out how to make
their tasks run in a specific order. Note that part of what makes Gulp fast and 
powerful*is* the ability to run multiple tasks asynchronously, so only use this
feature if you actually need it.</section>

 [1]: http://gulpjs.com/
 [2]: http://gruntjs.com/
 [3]: https://github.com/kriskowal/q