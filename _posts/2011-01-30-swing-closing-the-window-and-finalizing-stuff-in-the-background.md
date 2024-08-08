---
title: 'Swing - Closing the window and finalizing stuff in the background'
author: yossale

 
categories:
  - Programming
  - Swing
---
I'm writing a program that is a little data intensive. At first , when the user clicked the X and choose"yes" at the"are you sure" box , I would do the clean up and then close the window . The problem was that the clean up sometimes took up to 1 minute , in which the window was frozen and annoyingly stuck on screen. 

I've tried several different approaches to make it close the window and keep the cleaning in the background , but nothing worked.  
Why?

Because I used 

```java
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```

instead of 

```java
this.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
```

Let's see some code: 

```java
public class MyFrame extends JFrame {

		public MyFrame() {
			super("My Frame");			
			this.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);

			this.addWindowListener(new WindowAdapter() {						                                       


				//This is the function that will be invoked when the user clicks on the close button (the X)
				//At the end of this function , there will be no window on the screen , because we set in the 
				//previous line the frame's defautl close operation to DisposeOnClose				

				@Override                               
				public void windowClosing(WindowEvent e) {
					int confirmed = JOptionPane.showConfirmDialog(null,
							"Are you sure you want to exit?",
							"Leaving so soon?", JOptionPane.YES_NO_OPTION);
					if (confirmed == JOptionPane.YES_OPTION) {
						System.out.println("The windows is no longer visible");
                        dispose();
                                                
					}
				}

				/*
				This is the function that is invoked when the window is closed (i.e , immediatly after the previous 
				function "windowClosing" exits). 
                                If the frame's default closing operation was "EXIT_ON_CLOSE" , this function wouldn't run.
				*/
				@Override
				public void windowClosed(WindowEvent e) {								
					viewManager.cleanUpTheMess();
				}
			});

		}
	}

```

What would have happened if we used EXIT\_ON\_CLOSE instead of DO\_NOTHING\_ON_CLOSE?  
Simple - the"windowClosed" function wouldn't run . It's as if there was a System.exit(0) at the end of the windowClosing function.