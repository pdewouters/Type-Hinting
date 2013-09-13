I created this document as a way to gain more understanding about the use of type hinting in PHP methods.

In a project I was working on recently, the Docblocks and function signature looked like this:

	/**
	 * Our object constructor
	 * @param array $myArr
	 */
	public function __construct( $myArr = array() ) {

        // Initialize our property to the value of the passed param
		$this->objArr = $myArr;

		// call a method to test this code
		$this->print_myarr();
	}

This function accepts an `$options` parameter which is optional, and will be an empty array if not provided. But you could still pass any other variable type, and get errors or unexpected results.

A more robust solution is to enforce the `array` type on the parameter, by using Type Hinting: `array $options = array()`. Calling this function and passing a value other than an array in the `$option` parameter will cause a PHP Catchable fatal error.

	/**
	 * Our object constructor
	 * @param array $myArr
	 */
	public function __construct( array $myArr = array() ) {

        // Initialize our property to the value of the passed param
		$this->objArr = $myArr;

		// call a method to test this code
		$this->print_myarr();
	}

The next step is to catch that error, when instantiating a new object:

	try {

		$my_string_var = 'I am a string';
		$myObj1 = new Arr_Test( $my_string_var );

	} catch (Exception $e) {

		echo $e->getMessage();

	}

In the above example, we're passing a string value as a parameter to the constructor. This will result in a fatal error. The try…catch block will have no effect! Why? It's up to us to write a custom error handler to catch the Exception - [Stackoverflow answer](http://stackoverflow.com/a/2468534/285564 "go to stackoverflow")

	/**
 	 * Custom Error Handler for E_RECOVERABLE_ERROR
	 *
	 * http://stackoverflow.com/a/2468534/285564
	 *
	 * @param $errno
	 * @param $errstr
	 * @param $errfile
 	 * @param $errline
 	 * @return bool
	 * @throws ErrorException
 	*/
	function myErrorHandler( $errno, $errstr, $errfile, $errline ) {

		if ( E_RECOVERABLE_ERROR===$errno ) {

			echo "'catched' catchable fatal error\n";

    			throw new ErrorException( $errstr, $errno, 0, $errfile, $errline );

    			// return true;
 	 	}
 	 return false;

	}

	// Set up the custom error handler
	set_error_handler( 'myErrorHandler' );
	…
	
We can now use try…catch blocks to handle the Exception

	try {

		$my_string_var = 'I am a string';
		$myObj1 = new Arr_Test( $my_string_var );

	} catch (Exception $e) {

		echo $e->getMessage();

	}
	
Here's a complete example:

	<?php

	/**
 	 * Custom Error Handler for E_RECOVERABLE_ERROR
	 *
	 * http://stackoverflow.com/a/2468534/285564
 	 *
 	 * @param $errno
	 * @param $errstr
	 * @param $errfile
	 * @param $errline
	 * @return bool
 	* @throws ErrorException
 	*/
	function myErrorHandler( $errno, $errstr, $errfile, $errline ) {

		if ( E_RECOVERABLE_ERROR===$errno ) {

    			echo "'catched' catchable fatal error\n";

   			 throw new ErrorException( $errstr, $errno, 0, $errfile, $errline );

    			// return true;
  		}
 		 return false;

	}

	// Set up the custom error handler
	set_error_handler( 'myErrorHandler' );

	/**
 	* Class Arr_Test
 	*/
	class Arr_Test {

		/**
	 	* Gives us a property to play with
	 	* @var array
		*/
		var $objArr = array();

		/**
		 * Our object constructor
		 * @param array $myArr
		 */
		public function __construct( array $myArr = array() ) {

        		// Initialize our property to the value of the passed param
			$this->objArr = $myArr;

			// call a method to test this code
			$this->print_myarr();
		}

		/**
	 	* Simply iterates an array
	 	*/
		protected function print_myarr(){

			if ( count( $this->objArr ) > 0 ) {

				for ( $i=0; $i < count( $this->objArr ); $i++) {

					echo $this->objArr[$i]; // this would fail if our property is not an array

				}

			} else {
				echo "objArr is empty";
			}			

		}

	}

	try {

    		$my_string_var = 'I am a string';
		$myObj1 = new Arr_Test( $my_string_var );

	} catch (Exception $e) {

		echo $e->getMessage();

	}

A few code samples with the different cases:

[No try…catch and no error handler](https://gist.github.com/pdewouters/6550105)

[try…catch blocks but no custom error handler](https://gist.github.com/pdewouters/6550083)

[No type hinting](https://gist.github.com/pdewouters/6550033)

[Type hinting but optional parameter](https://gist.github.com/pdewouters/6550014)

[catching type hinting errors correctly](https://gist.github.com/pdewouters/6549934)

[complete example with correct param type, no errors](https://gist.github.com/pdewouters/6549885)

