.. highlight:: java

Swing JavaBuilder in 60 seconds or less
=======================================

Here's a sample of what you can do with the Swing JavaBuilder in 60 seconds or less. Hopefully, it will
make it clear as to what the productivity benefits are.

To show off its abilities we will create a simple app that prompts for a person's data and simulates saving
it to a database via a long running task on a background thread.

1. Download the latest Swing JavaBuilder ZIP from http://javabuilders.org
   
2. In Eclipse, create a new Java project called "PersonApp" and create a default package "person.app"
       
3. Add the Swing JavaBuilder jar and all of its dependencies (from the "/lib" folder) to the project's build path
       
4. Create the *person.app.Person* class that will represent our model::

       package person.app;
       
       import java.text.MessageFormat;
       
       public class Person {
                private String firstName;
                private String lastName;
                private String emailAddress;
                /**
                 * @return the firstName
                 */
                public String getFirstName() {
                         return firstName;
                }
                /**
                 * @param firstName the firstName to set
                 */
                public void setFirstName(String firstName) {
                         this.firstName = firstName;
                }
                /**
                 * @return the lastName
                 */
                public String getLastName() {
                         return lastName;
                }
                /**
                 * @param lastName the lastName to set
                 */
                public void setLastName(String lastName) {
                         this.lastName = lastName;
                }
                /**
                 * @return the emailAddress
                 */
                public String getEmailAddress() {
                         return emailAddress;
                }
                /**
                 * @param emailAddress the emailAddress to set
                 */
                public void setEmailAddress(String emailAddress) {
                         this.emailAddress = emailAddress;
                }
                @Override
                public String toString() {
                    return MessageFormat.format(
                        "{0} {1} : {2}", getFirstName(), 
                        getLastName(),
                        getEmailAddress());
                }
       }
       
5. Create a *PersonApp.properties* file in the root package with the internationalized resources::

    button.save=Save
    button.cancel=Cancel
    label.firstName=First Name:
    label.lastName=Last Name:
    label.email=Email
    frame.title=Enter Person Data
    
6. Create the view YAML file *PersonApp.yml* in the *person.app* package::

        JFrame(name=frame, title=frame.title, size=packed, defaultCloseOperation=exitOnClose):
            - JButton(name=save, text=button.save, onAction=[$validate,save,done])
            - JButton(name=cancel, text=button.cancel, onAction=[$confirm,cancel])
            - MigLayout: |
                  [pref]              [grow,100]     [pref]             [grow,100]
                  "label.firstName"   txtFirstName   "label.lastName"   txtLastName
                  "label.email"       txtEmail+*
                  >save+*=1,cancel=1
        bind:
            - txtFirstName.text: person.firstName
            - txtLastName.text: person.lastName
            - txtEmail.text: person.emailAddress
        validate:
            - txtFirstName.text: {mandatory: true, label: label.firstName}
            - txtLastName.text: {mandatory: true, label: label.lastName}
            - txtEmail.text: {mandatory: true, emailAddress: true, label: label.email}
           
7. Create the controller Java class *person.app.PersonApp* (same package where the YAML file is)::

    package person.app;
    
    import javax.swing.JFrame;
    import javax.swing.JOptionPane;
    import javax.swing.SwingUtilities;
    import javax.swing.UIManager;
    
    import org.javabuilders.BuildResult;
    import org.javabuilders.annotations.DoInBackground;
    import org.javabuilders.event.BackgroundEvent;
    import org.javabuilders.event.CancelStatus;
    import org.javabuilders.swing.SwingJavaBuilder;
    
    @SuppressWarnings({"serial","unused"})
    public class PersonApp extends JFrame {
    
        private Person person;
        private BuildResult result;
    
        public PersonApp() {
            person = new Person();
            person.setFirstName("John");
            person.setLastName("Smith");
            result = SwingJavaBuilder.build(this);
        }
    
        public Person getPerson() {
            return person;
        }
    
        private void cancel() {
            setVisible(false);
        }
    
        @DoInBackground(cancelable = true, 
            indeterminateProgress = false, progressStart = 1, 
            progressEnd = 100)
        private void save(BackgroundEvent evt) {
            // simulate a long running save to a database
            for (int i = 0; i < 100; i++) {
                // progress indicator
                evt.setProgressValue(i + 1);
                evt.setProgressMessage("" + i + "% done...");
                // check if cancel was requested
                if (evt.getCancelStatus() != CancelStatus.REQUESTED) {
                    // sleep
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                    }
                } else {
                    // cancel requested, let's abort
                    evt.setCancelStatus(CancelStatus.COMPLETED);
                    break;
                }
            }
        }
    
        // runs after successful save
        private void done() {
            JOptionPane.showMessageDialog(this, "Person data: " + person.toString());
        }
    
        /**
         * @param args
         */
        public static void main(String[] args) {
            SwingUtilities.invokeLater(new Runnable() {
                public void run() {
                    // activate internationalization
                    SwingJavaBuilder.getConfig().addResourceBundle("PersonApp");
                    try {
                        UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
                        new PersonApp().setVisible(true);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }

       
8. Run the PersonApp.main() method. You should see an input dialog like this appear:

.. image:: images/person-app.jpg

.. note::

    Notice that the default person name is propagated from the Java code to the UI via data binding.
    
    All the controls are created and the layout is executed without the need for an IDE-specific GUI builder.
    Also, many of the controls were auto-created without being explicitly defined. Putting a resource name
    within a String literal automatically created ``JLabel`` instances, while defining a field with a ``txt``
    prefix automatically created ``JTextField`` instances. All without any additional YAML or Java code.

The resource keys entered in quotes in the layout section have been used to automatically create JLabel(s) 
and populate their text with the value of the resource key.       
       
9. Enter an invalid email address for the person and press Save:

.. image:: images/person-app-validation.jpg
  
The validation logic (invoked via "$validate") executed and perform basic input validation.
   
10. Enter a valid email address:

.. image:: images/person-app-2.jpg

11. Press "Save". The save() Java method is executed (which simulates a long running database save with a progress bar) and 
    since it is annotated with the @DoInBackground annotation it will automatically run on a background thread 
    using the SwingWorker library.

.. image:: images/person-app-3.jpg

12. After the save logic executes, the done() Java method is executed to inform the user the save was successful. 
    Notice that the email address we entered was automatically propagated back to the underlying bean using databinding.

.. image:: images/person-app-4.jpg

13. Press 'Cancel' to close the window. Since you specified *"$confirm"* in the action handler, it will automatically 
    prompt the user to confirm the action. If they select "Yes", the *cancel()* Java  method will be called and the 
    window will close.
    
.. image:: images/person-app-5.jpg    
    
**Summary**

* 16 lines of YAML
* 3 simple Java methods to handle save(), done() and cancel() (and without any of the logic to create and layout the controls)

That is all we needed to create a fully functional application with control creation and layout, data input
validation and executing long running business methods on a background thread via SwingWorker. Not 
to mention it's fully localized with all the labels being automatically fetched from a ResourceBundle
