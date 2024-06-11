# Paws - ServiceNow Adoption App

## Overview
Paws is an application developed in ServiceNow Studio to facilitate dog adoptions from animal shelters. It leverages ServiceNow's low-code/no-code development capabilities to create a simple process for managing adoption requests.

![Smoke](https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/50bb5e6f-9ce0-4451-a7a5-a4ce576fd21a)
*Maxy, the lovable puppy!*

## Design Aspects

### Components:

#### Tables:
1. **Dogs (x_1108343_paws_dogs)**
   - This table stores information about dogs available for adoption.
   
     <img width="1185" alt="Screenshot 2024-06-10 at 10 15 13 PM" src="https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/5972b083-f27e-4956-a9a2-49fbbabc9752">

2. **Adoption Centers (x_1108343_paws_adoption_center)**
   - This table contains data about adoption centers where dogs are housed. <br>
   
     <img width="1194" alt="Screenshot 2024-06-10 at 10 15 59 PM" src="https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/c2eca32e-7435-4f19-b138-93d5e25bc9da">


#### UI Page:
- **Create Dog**
  - This UI page provides a form for adding a new dog to the system. It allows users to input information such as the name, age, and vaccination status of the dog. Upon submission of the form, an AJAX request is made to the server to create the dog record. Once the record is successfully created, a message is displayed to the user with the details of the newly created dog, including a link to view the dog's details.
  - **Components**:
    - **Form Fields**:
      - **Name of dog**: Text input field where users can enter the name of the dog.
      - **Age of dog**: Text input field where users can enter the age of the dog.
      - **Shots**: Checkbox indicating whether the dog has received its shots.
      - **Neutered**: Checkbox indicating whether the dog has been neutered.
    - **Submit Button**: Button labeled "Add" that triggers the submission of the form.
    - **Message Display**: A message area where success or error messages are displayed to the user after submitting the form.
      
![Create Dog UI Page](https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/79d313b7-6642-4030-a929-df3160a128a3)
*Create Dog UI Page: This image represents the UI page where users can create a new dog record.*

##### UI Action:
- **Adopt**
  - The `adoptDog()` function prompts the user to enter their email address. If a non-empty email address is provided, it creates a new instance of `GlideAjax` to handle the AJAX request. Parameters such as the adoption center and adopter's email address are added to the request. The `getXMLAnswer()` function is then called to send the request to the server. If the email address is empty, an alert notifies the user that the entered email address appears to be invalid. The `ajaxProcessor()` function is called when the server responds to the AJAX request. It displays an alert message thanking the user for their request and informing them that someone will be with them shortly. <br><br>
    
   ```javascript
    function adoptDog() {
    var email = prompt('Please enter your email address');
    if (email != '') {
        var ga = new GlideAjax('fetchUtils');
        ga.addParam('sysparm_name', 'createEmailNotification');
        ga.addParam('sysparm_adoption_center', g_form.getValue('adoption_center'));
        ga.addParam('sysparm_adopter_email', email);
        ga.getXMLAnswer(ajaxProcessor);
    } else {
        alert('The email address you entered does not appear to be valid.');
    }
    }
    function ajaxProcessor(answer) {
    alert('Thank you for your request. Someone will be with you shortly.');
    }
    ```

#### Script Includes:
- **fetchUtils**
  - The `fetchUtils` script include contains reusable functions for handling AJAX requests and processing data related to dog adoption in the Paws application.
    - **createDog()**: This function handles the creation of a new dog record. It retrieves parameters passed in an AJAX request, such as the dog's name, age, and vaccination status. Then, it creates a new GlideRecord object for the 'x_1108343_paws_dogs' table, sets its fields based on the parameters, inserts the new record, and returns details about the newly created dog record as a concatenated string.
    - **createEmailNotification()**: This function is responsible for sending email notifications related to dog adoptions. It retrieves the adopter's email address and the adoption center's sys_id from parameters passed in an AJAX request. Then, it retrieves the email address of the adoption center from the corresponding GlideRecord object, and queues an event (`x_1108343_paws.adoption_email`) to send the email notification. <br><br>
    
      ```javascript
      var fetchUtils = Class.create();
      fetchUtils.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
      createDog: function() {
        var dogName = this.getParameter('sysparm_dog_name');
        var dogAge = this.getParameter('sysparm_dog_age');
        var dogShots = this.getParameter('sysparm_dog_shots') === 'true';
        var dogNeutered = this.getParameter('sysparm_dog_neutered') === 'true';

        // insert dog
        var newDog = new GlideRecord('x_1108343_paws_dogs');
        newDog.newRecord();
        newDog.name = dogName;
        newDog.age = dogAge;
        if (dogShots) {
            newDog.shots = true;
        }
        if (dogNeutered) {
            newDog.neutered = true;
        }

        var sysID = newDog.insert();
        var dogNumber = newDog.number.getDisplayValue();
        var dogLink = newDog.getLink();

        // return values
        return dogName + '|' + dogNumber + '|' + dogLink;
      },
      createEmailNotification: function() {
        var adopterEmail = this.getParameter('sysparm_adopter_email');
        var adoptionCenter = this.getParameter('sysparm_adoption_center');
        var adoptionCenterEmail = '';
        var ac = new GlideRecord('x_1108343_paws_adoption_center');
        ac.get(adoptionCenter);
        adoptionCenterEmail = ac.email.getDisplayValue();
        gs.eventQueue('x_1108343_paws.adoption_email', ac, adoptionCenterEmail, adopterEmail);
        return;
      },
      type: 'fetchUtils'});
      ```
##### Event:
- **adoption_email**
  - This event registration name is associated with the process of sending email notifications related to dog adoptions. It triggers the execution of specific functions or workflows within the application that handle the sending of adoption-related emails to adopters, adoption centers, or other relevant parties.
    ![Event](https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/fbc4bc9a-4e90-4fce-947f-6b6bebd6c66f")
*Event name x_1108343_paws.adoption_email*

##### Email Notification:
- **Adopt Dog Email**
  - This email notification type is triggered by the app to notify users about successful dog adoptions. It sends an email to adopters confirming their adoption request.
   ![Email Notification](https://github.com/shackerica/Paws-ServiceNow-App/assets/19885127/2e827e4e-4927-4319-adbe-6672eb76123f)
*Notification Adopt Dog*

#### Demo:
- The Paws ServiceNow Adoption App streamlines the process of dog adoptions, providing a user-friendly interface for both users and adoption centers. With features like creating new dog records, adopting dogs, and sending email notifications, Paws simplifies the adoption process, making it efficient and enjoyable for all parties involved.
  https://www.loom.com/share/e7a0c39f95aa43c0aba0b7a157afb7f8?sid=86e673d3-5792-4ad5-931b-3bdbd50f1d47

## Skills Used
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![jQuery](https://img.shields.io/badge/jQuery-0769AD?style=flat&logo=jquery&logoColor=white)
![ServiceNow](https://img.shields.io/badge/ServiceNow-5CB85C?style=flat&logo=servicenow&logoColor=white)

