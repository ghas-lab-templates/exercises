
# Module 2 - Secret Scanning

## Enablement  

### Lab 1 - Setting Up a Custom Security Configuration and Enabling Secret Scanning

#### Objective 
In this lab, you will learn how to create and apply a custom security configuration to repositories in your organization. 

#### Steps

1. Click on your Organization's settings. In the `Security` section of the sidebar, select the `Code security` dropdown menu, then click `Configurations`.  You will be navigated to the `Code security configurations` page, click `New configuration` button.
<details>
  <summary> Animated Guide</summary>

![alt text](images/new-config.gif)

</details>

2. To help identify your custom security configuration and clarify its purpose, name your configuration and create a description. 

<details>
  <summary>Need Help? View Configuration Screenshot</summary>
  
![alt text](images/confignameanddesc.png)

</details>

3. For each configuration option select the following:
 - `GitHub Advanced Security Features`: Select `Include`.
 - `Secret Scanning - Alerts`: Select `Enabled`.
 - All Other Settings: Select `Not set`.
 - In the `Policy` options, for `Use as default for newly created repositories`, select `All repositories`.
 - In the `Policy` options, for `Enforce Configuration`, select `Don't Enforce`.

4. Click on `Save Configuration` button. Please confirm save if prompted.

  <details>
 <summary>Need Help? View Configuration Screenshot</summary>  
   
   ![alt text](images/config-options-secret-scanning.png)
   
 </details>

5. The page will redirected to the Configurations page. Click on the Apply to dropdown and select All repositories. There will be a prompt for confirmation, select Apply

<details>
  <summary>Animated Guide</summary>
  
**TO DO: GIF THIS WHEN TOTAL NUMBER OF REPOS CONFIRMED**

![alt text](images/applytoallrepos.png)
</details>

### Lab 2 - Cutom Patterns with AI 

#### Objective

The objective of this lab is to demonstrate the uage of cutom patterns in identifying secrets which are not part of the Secret Scanning Partner Program in GitHub

#### Steps
1. Navigate to the `mona-gallery` repository in your GitHub Organization
2. Please note that the Custom patterns can be created at 3 levels of Hierarchy - Enterprise, Organisation & Repository
3. For this Lab, we will create the custom pattern in the repo scope
4. Navigate to Settings tab of the repository, click on **Code security** section, and under Custom Patterns click on New pattern
5. In the top right hand corner, click on Generate with AI

<details>
  <summary> Animated Guide</summary>

![alt text](images/custom-pattern-with-ai.gif)

</details>

6. In this lab we have a custom secret that has been embedded in the following location `storage/src/main/resources/application.properties`
7. Specifically we will try to take AI assitance to create a regular expresssion for the minio.password mapped to `mona_value_abc124`
8. Fill in the options `I want a regular expression that` and `Examples of what I am looking for`
9. Once you are happy with th generated regular expression, click on the buttom `Use result`
10. This will copy the generated regular expresssion and the test strings into the New Custom Pattern creation page


<details>
  <summary> Animated Guide</summary>

![alt text](images/custom-pattern-prompt.gif)

</details>

11. Give an appropriate `Pattern name` and click on `Save and dry run`

<details>
  <summary> Animated Guide</summary>

![alt text](images/custom-pattern-publish.gif)

</details>

### Lab 3 - Push Protection 

#### Objective

The objective of this Lab is to demonstrate and familiarize the the participants with the Secret Scanning Push protection feature

#### Steps
1. Navigate to the `mona-gallery` repository in your GitHub Organization
2. Continuing from Lab 2, where we already created a custom pattern to identify a custom secret and published the custom pattern which captured the `mona_value_abc124` secret
3. In this exercies, we will now enable the Push protection feature at the repository level (push protection can also be enabled at the Enterprise and Organiation level) and then psecifically enable push protection for the newly created custom pattern
4. Oncel, push protection is enabled for the custom pattern, we will then try to commit some changes in the source code by trying to commit a string which matches the custom pattern
5. Since push protection is enabled for thi repo, the commit will be blocked for the user trying to commit the code

<details>
  <summary> Animated Guide</summary>

![alt text](images/custom-pattern-publish.gif)

</details>
