# wordpress-sample
Sample code for wordpress

Issuer API integration 
-----------------------

1)	Set up environment file 
	Create a file (e.g., .env.php) in your website root directory. This file will store API credentials, URLs, and other sensitive information. Place this file outside the public directory to prevent direct access. Copy the below content in the file and save it.
	If a similar file already exists, add these values to that file and save.

	<?php
	// .env.php

	// API URLs
	define('API_GET_URL', 'https://console-api-uat.diceid.com/v1/api-token');
	define('API_POST_URL', 'https://console-api-uat.diceid.com/v1/cred/issuance');

	// API credentials
	define('x-api-key', '<your x_api_key>');
	define('org_id', '<your Org_id>');
	define('invoked_by', '<your User_id>');
	define('schema_name', '<Schema_name>');
	define('schema_version', '<Schema_version>');
	define('template_id', '<Certificate_template_id>');
	?>

2)	Include the above environment file in your WordPress theme or plugin: 
	In your WordPress theme's functions.php file or in your custom plugin, include the .env.php file with full path:

	// functions.php or your custom plugin file
	include_once('/path/to/your/.env.php');
	
3)  Create a function to process the GET and POST API response
	Add the below function in your functions.php file to execute the GET and POST API functions.

	// functions.php or your custom plugin file	

	function process_api_response() {	
		$response_get = make_get_api_call(); 

		if ($response_get !== false) {
			// Process the response body
			// Parse the JSON response
			$data = json_decode($response_get);

			// Check if decoding was successful
			if ($data === null) {
				// Handle JSON decoding error
				echo "Bearer Token Generation API Call Error";
				return false;
			}	
			// Now you can traverse the JSON data using standard PHP array/object syntax	
			// Accessing data from an object
			$status = $data->status;
			$result = $data->result;
			$token = $result->token;

			// Process the retrieved data as needed
			// Call the next api -> credential issuance api and pass the necessary arguments
			$response_post = make_post_api_call($token); 
			
			if ($response_post !== false) {
				// Parse the JSON response
				$data = json_decode($response_post);

				// Check if decoding was successful
				if ($data === null) {
					// Handle JSON decoding error
					echo "Credential Issuance API Call Error";
					return false;
				}				
			} else {
				// Handle API call errors
				echo "Credential Issuance API Call Error";
				return false;
			}					
		} else {
			// Handle API call errors
			echo "Bearer Token Generation API Call Error";
			return false;
		}	
	}

4)	Create a function to make GET API calls. 
	Add the below function in your functions.php file to get Bearer Token.

	function make_get_api_call() {
		// Build GET request headers
		$headers = array(
			'x-api-key' => x-api-key,
			'org_id' => org_id,
			'invoked_by' => invoked_by
		);

		// Build GET request URL
		$url = API_GET_URL

		// Make the API request
		$response = wp_remote_get($url, array('headers' => $headers));

		// Check for errors and process the response
		if (is_array($response) && !is_wp_error($response)) {
			$response_body = wp_remote_retrieve_body($response);
			// Process the response data ($response_body) as per your requirement
			echo "API Call Successful. Bearer Token Issued.";			
			return $response_body;
		} else {
			if (is_wp_error($response)) {
				$error_message = $response->get_error_message();
				// Handle API call error
				echo "API Call Error: $error_message";
				return false;
			}		
			$response_code = wp_remote_retrieve_response_code($response);
			if ($response_code !== 200) {
				// Handle non-200 status codes as per your requirement
				echo "API Call Error: Received status code $response_code";
				return false;
			}
		}
	}
	
	
5)	Create a function to make POST API calls . 
	Add the below function in your functions.php file to invoke the Credential Issuance API.	

	function make_post_api_call(token) {
		// Build POST request headers
		$headers = array(
			'Authorization' => 'Bearer ' . $token,
			'x-api-key' => x-api-key,
			'org_id' => org_id,
			'invoked_by' => invoked_by,
			'Content-Type' => 'application/json'
		);
		
		// Varibale Data -- can be captured from client api response and passed here as input OR read from external file
		$cert_attribute = array(
			'Name' => 'Rohan',
			'Skill' => 'java',
			'Date' => '15-Feb-2023',
			'Valid Till' => '31-Jan-2025'
			// Add other parameters as needed
		);
		
		$email_attribute = array(
			'invitee_email' => 'testemail@gmail.com'
		);
		
			
		// Build POST request body
		$data = array(
			'invite_modes' => array('email'),
			'invitee_name' => 'Rohan',		
			'dice_display_name' => 'Cert for Java Credential',	
			'attributes' => cert_attribute,			
			'schema_name' => schema_name,
			'schema_version' => schema_version,
			'is_cert_required' => true,			
			'template_id' => template_id,
			'email' => email_attribute,
			// Add other POST parameters as needed
		);

		// Make the API request
		$response = wp_remote_post(API_POST_URL, array('headers' => $headers, 'body' => $data));

		// Check for errors and process the response
		if (is_array($response) && !is_wp_error($response)) {
			$response_body = wp_remote_retrieve_body($response);
			// Process the response data ($response_body) as per your requirement
			echo "API Call Successful. Credential Issued.";
			return $response_body;
		} else {
			if (is_wp_error($response)) {
				$error_message = $response->get_error_message();
				// Handle API call error
				echo "API Call Error: $error_message";
				return false;
			}		
			$response_code = wp_remote_retrieve_response_code($response);
			if ($response_code !== 200) {
				// Handle non-200 status codes as per your requirement
				echo "API Call Error: Received status code $response_code";
				return false;
			}		
		}
	}

6)	Call the API functions when needed: 
	Now, you can call the make_get_api_call() and make_post_api_call() functions from anywhere within your theme files or custom plugins.
	An exapmle is shown by calling from function process_api_response().

	For example, you might trigger the API calls when a user submits a form, in a custom shortcode, or as part of a scheduled task using WordPress hooks.

	Keep in mind that this is a basic implementation, and you may need to add additional error handling, validation, and security measures depending on your specific use case. Also, ensure that you properly secure the env.php file and restrict access to it to avoid exposing sensitive information. Replace sample data to match your actual data wherever used.
	
	Keep in mind that the specific structure of the JSON response will depend on the API you are using. So, adjust the code accordingly to match the JSON structure of your API response.
	
	Remember that error handling can be customized based on the specific requirements of your application and the API you are working with. You can add more elaborate error messages, logging, and recovery mechanisms depending on your needs. The error handling given here is just echoing messages. Please elaborate as per your requirement.	

	Additionally, always check for errors while parsing the JSON response with json_decode(). If the JSON is invalid, the function will return null, and you should handle the error appropriately.
