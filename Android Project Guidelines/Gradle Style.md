# 3. Gradle Style

## 3.1 Dependencies

### 3.1.1 Versioning

Where applicable, versioning that is shared across multiple dependencies should be defined as a variable within the dependencies scope. For example:


    final SUPPORT_LIBRARY_VERSION = '23.4.0'

    compile "com.android.support:support-v4:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:recyclerview-v7:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:support-annotations:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:design:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:percent:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:customtabs:$SUPPORT_LIBRARY_VERSION"

This makes it easy to update dependencies in the future as we only need to change the version number once for multiple dependencies.

### 3.1.2 Grouping

Where applicable, dependencies should be grouped by package name, with spaces in-between the groups. For example:


    compile "com.android.support:percent:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:customtabs:$SUPPORT_LIBRARY_VERSION"

    compile 'io.reactivex:rxandroid:1.2.0'
    compile 'io.reactivex:rxjava:1.1.5'

    compile 'com.jakewharton:butterknife:7.0.1'
    compile 'com.jakewharton.timber:timber:4.1.2'

    compile 'com.github.bumptech.glide:glide:3.7.0'


`compile` , `testCompile` and `androidTestCompile`  dependencies should also be grouped into their corresponding section. For example:


    // App Dependencies
    compile "com.android.support:support-v4:$SUPPORT_LIBRARY_VERSION"
    compile "com.android.support:recyclerview-v7:$SUPPORT_LIBRARY_VERSION"

    // Instrumentation test dependencies
    androidTestCompile "com.android.support:support-annotations:$SUPPORT_LIBRARY_VERSION"

    // Unit tests dependencies
    testCompile 'org.robolectric:robolectric:3.0'

Both of these approaches makes it easy to locate specific dependencies when required as it keeps dependency declarations both clean and tidy 🙌


### 3.1.3 Independent Dependencies

Where dependencies are only used individually for application or test purposes, be sure to only compile them using `compile` , `testCompile` or `androidTestCompile` . For example, where the robolectric dependency is only required for unit tests, it should be added using:


    testCompile 'org.robolectric:robolectric:3.0'
