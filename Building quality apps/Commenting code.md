Comments are a part of the documentation for the code, and they should be written and maintained with great care.

Have you ever stared at a piece of code and screamed `WHY? WHY?!? WHY WOULD YOU DO THAT?` at the screen?

Follow the advice provided here, and your colleagues won't have to scream and will love working with you.

## Avoid useless comments

Comments that do not add any value to the code should be removed, and naming should be improved where possible.

### BAD

```java
int h; // days since refresh

clearCache(); // clears the cache
```

### GOOD

```java
int daysSinceRefresh;

clearCache();
```

## Comment the why, not the how

[Code Tells You How, Comments Tell You Why](http://blog.codinghorror.com/code-tells-you-how-comments-tell-you-why/)
(blog post by Jeff Atwood, co-founder of [StackOverflow](http://stackoverflow.com/))

## Having no comments is better than having misleading comments

Having no comments at all is better than having comments that are false or misleading.

If you change a commented piece of code so that the comment no longer holds true, be sure to update the comment to reflect the new state of the code.
Comments, like code, need to be maintained.

## Examples of good comments

```java
@Override
public void onDeviceMetadataObtained(DeviceAuthMetadata deviceAuthMetadata) {
    /**
     * 'resend code' during 2fa enrollment will create a new device
     * so, we need to save the new metadata so that we can send the correct followOnId and deviceId
     */
    this.deviceAuthMetadata = deviceAuthMetadata;
    view.hideActionLoadingDialog();
}
```

```java
// always construct a new callback because we can have multiple requests running in parallel
calculationListener = new CancelableCallback() {
    @Override
    public void onSuccess(ComparisonQuoteResponse comparisonQuoteResponse, Response response) {
        if (!isCanceled) {
            CalculatorPresenterImpl.this.onSuccess(comparisonQuoteResponse, false);
        }
    }
};
```

## Comments should always be appropriate

We all sometimes get frustrated in the development process, and it can be tempting to vent those frustrations in the code.

However, the code and comments should be *safe for work* and should not be offensive because the code often gets delivered to the client as a part of the project. The client may not share your sense of humor, and you don't want to spend hours making comments safe for work.
