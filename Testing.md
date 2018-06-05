Writing tests is not the most glamorous part of developing an Android application but it is an invaluable one. There are two main types of tests for Android apps - unit tests and instrumentation tests.

Unit tests test small components in isolation while instrumentation tests test larger functionality like whole screens or even multiple screens.

## Unit tests

Unit tests are easy to write and fast to execute since they don't need an Android device to run - they run on a plain JVM.

This means that the code needs to be cleanly separated so we can write useful unit tests for it.

Usually it's very useful to test the presenter logic, assuming MVP is used in the app (see [MVP](/MVP.md) chapter about details).
This means that presenters need to be Android-agnostic.

In general, the most useful tests cover the business logic of the app and the most important functionalities, paying special attention to edge cases.

### Example

The following method tests if the ticket barcode conforms to the specified format.

```java
/**
 * @param code the ticket code from a scanned ticket
 * @return true if the ticket code is valid
 */
public static boolean isTicketCodeValid(@NonNull String code) {
    if (code.length() != TICKET_BARCODE_LENGTH) {
        return false;
    }

    int[] digits = new int[code.length()];

    for (int i = 0, length = code.length(); i < length; i++) {
        if (!Character.isDigit(code.charAt(i))) {
            // all the characters need to be digits
            return false;
        }
        digits[i] = Character.digit(code.charAt(i), 10);
    }

    int controlDigit = 0;
    for (int i = digits.length - 2; i > -1; i = i - 2) {
        controlDigit += digits[i];
    }
    controlDigit = 3 * controlDigit;
    for (int i = digits.length - 3; i > -1; i = i - 2) {
        controlDigit += digits[i];
    }
    controlDigit = 10 - controlDigit % 10;
    controlDigit = controlDigit < 10 ? controlDigit : 0;
    return controlDigit == digits[digits.length - 1];
}
```

And these are the tests for the method:

```kotlin
class TicketCodeUtilTest {

    @Test
    fun shouldRecognizeCorrectTicketCode() {
        assertThat(TicketCodeUtil.isTicketCodeValid("0107086164038960515268")).isTrue()
    }

    @Test
    fun shouldRecognizeIncorrectTicketCode() {
        assertThat(TicketCodeUtil.isTicketCodeValid("0107086164038960515260")).isFalse()
    }

    @Test
    fun shouldRecognizeIncorrectTicketCodeIfNotAllDigits() {
        assertThat(TicketCodeUtil.isTicketCodeValid("010708616403896051526A")).isFalse()
    }

    @Test
    fun shouldRecognizeIncorrectTicketCodeIfLessThan22() {
        assertThat(TicketCodeUtil.isTicketCodeValid("010708616403896051526")).isFalse()
    }

    @Test
    fun shouldRecognizeIncorrectTicketCodeIfMoreThan22() {
        assertThat(TicketCodeUtil.isTicketCodeValid("01070861640389605152680")).isFalse()
    }

    @Test
    fun shouldRecognizeIncorrectTicketCodeIfEmpty() {
        assertThat(TicketCodeUtil.isTicketCodeValid("")).isFalse()
    }
}
```

Notice that these tests don't just test the *happy path*, they also test edge cases where the input isn't completely numeric or of the required length.

### Libraries

These libraries make it easier to write unit tests:

- `junit:junit:4.12` - a simple annotation-based test framework
- `org.assertj:assertj-core:3.6.0` - provides [readable assertion methods](http://joel-costigliola.github.io/assertj/)

As seen in the example above, we like to write tests in Kotlin because it's less verbose than Java and often leads to smaller, more readable tests.

## Instrumentation tests

Instrumentation tests cover more functionality and run on a real Android device (or emulator). Although they take more time to write and execute slower, they test whole features of the app and are also quite useful.

These tests can be written manually using the [Espresso test framework](https://developer.android.com/topic/libraries/testing-support-library/index.html#Espresso) or recorded using [Espresso test recorder](https://developer.android.com/studio/test/espresso-test-recorder.html).

For these kind of test you usually want to prepare a mock web server so the tests don't depend on the availability and state of a real test/staging API.
See below for more info on how to do that.

## Common testing techniques

### Building complex objects by deserializing json files

When testing some functionality requires you to build a complex object with many nested objects, you can make your life easier by placing a json representation of that object into the test resources and deserializing it at the beginning of the test.

This will decrease the test code and make it more readable.

See the `ResourceUtils` class below for a useful helper class.

### Parametrized tests

If you need to test a functionality for many different input and output pairs, you can write a [Parametrized JUnit test](https://github.com/junit-team/junit4/wiki/Parameterized-tests).

Example:

```kotlin
@RunWith(Parameterized::class)
class FullSerializationTest(val responseFilename: String) {

    companion object {

        @JvmStatic
        @Parameterized.Parameters(name = "{0}")
        fun responseFilenames(): Array<Any> {
            return listOf(
                    "1000000000000000000001",
                    "1000000000000000000002",
                    "1000000000000000000003",
                    "2000000000000000000001",
                    "2000000000000000000002",
                    "2000000000000000000003",
                    "3000000000000000000001",
                    "3000000000000000000002",
                    "3000000000000000000003"
            )
                    .map { "slip-responses/$it.json" }
                    .toTypedArray()
        }
    }

    /**
     * We test if the response we got from the API can be saved to the db and retrieved
     * without losing any information or failing for all the responses we have.
     */
    @Test
    fun serializationAndDeserialization() {
        val response = ResourceUtils.readFromFile(responseFilename)
        val downloadedGameTicket = SerializationUtil.deserializeGameTicketResponse(response)

        val databaseTicket = SerializationUtil.toDbModel(downloadedGameTicket)
        val databaseGameTicket = SerializationUtil.fromDbModel(databaseTicket)

        assertThat(databaseGameTicket).isEqualTo(downloadedGameTicket)
    }
}
```

The above test class will run the same test for each of the 9 input strings.

You may also find the [Burst library](https://github.com/square/burst) useful.

### Using a mock web server to simulate API

When testing, we don't want to execute real API calls. Instead, we use mock server and specify each response so we can test different use cases. Usually good idea is to provide OkHttp's [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) with Dagger. Then you have to start it and shutdown it before and after each test.

In each test you may want to enqueue response(s). Enqueuing works by [FIFO](https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)))

To enqueue response, first you have to create one and put it in, let's say `/resources/mockdata`, folder in androidTest flavor. Then you need to create utils class for reading from resources file. It may look like something like this:

```java
import java.io.InputStream;
import java.util.Scanner;

/**
 * Utility methods for accessing resources bundled with test APK. Standard Android Resources don't seem to work for test APK
 * (unable to fetch R.java).
 * <p>
 * Resources should be placed under /resources/mockdata folder in androidTest flavour. Use {@link #readFromFile(String)} to read a text
 * file to String giving only a name of the file located in /resources/mockdata folder.
 */
public class ResourceUtils {

    private static final String MOCK_DATA_DIRECTORY = "mockdata/%s";

    private ResourceUtils() {
    }

    /**
     * Converts InputStream to String.
     */
    public static String convertStreamToString(InputStream is) {
        Scanner s = new Scanner(is, "UTF-8").useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }

    /**
     * Reads a resource file to <code>String</code>.
     */
    public static String readFromFile(String filename) {
        InputStream is = ResourceUtils.class.getClassLoader().getResourceAsStream(String.format(MOCK_DATA_DIRECTORY, filename));
        return convertStreamToString(is);
    }
}
```

Once you have that, you can simply enqueue your next `200` server response by putting this in your test:

```java
String body = ResourceUtils.readFromFile(filename);
MockResponse mockResponse = new MockResponse().setBody(body).setResponseCode(HttpURLConnection.HTTP_OK);
mockWebServer.enqueue(mockResponse);
```

and next API call will result with that response.

#### Redirect responses

When you want to test redirection you have to be careful how you set URL in the Location header. When you are creating the URL you must set MockServer's host and port in the URL scheme. The code below is an example of a test that successfully tests redirection.

```java
@Test
public void mobileDataUserLowBudget() throws Exception {

    //First response with Location header and 302 code
    MockResponse mockResponse = new MockResponse()
            .setBody("")
            .setResponseCode(HttpURLConnection.HTTP_MOVED_TEMP)
            .setHeader("Location", "http://" + mockWebServer.getHostName() + ":" + mockWebServer.getPort() + "/molimo-vas-dopunite-kredit");
    enqueueResponse(mockResponse);

    //Second response with empty body and 200 code
    enqueueEmptyResponse(HttpURLConnection.HTTP_OK);
    startActivity();

    //Check that request exists and that it is made to proper URL
    RecordedRequest request = takeLastRequest();
    Assert.assertNotNull(request);
    Assert.assertTrue(request.getPath().contains("molimo-vas-dopunite-kredit"));
}
```

To properly test redirection you must:

*   Provide 2 responses because first response redirects you and second request is instantly executed, thus you receive 2nd response in your callback.  
*   Set **Location** header in 1st response ([Location header](https://en.wikipedia.org/wiki/HTTP_location)).
*   **DO NOT HARDCODE** Location header. If you hardcode URL whose host is different than MockServer's host and port, you will receive 404 error response and you will never receive the 2nd response that you provided. To successfully execute redirected request, you **MUST** get MockServer host and port and use them as host in your URL scheme!
*   Fetch last request and check if the request's URL is the one you provided in Location header.
*   Voila!
