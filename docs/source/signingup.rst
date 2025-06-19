Signing up to Viking
=====


Creating an account
------------

Follow the instructions on `Creating Accounts <https://vikingdocs.york.ac.uk/getting_started/creating_accounts.html>`_.

Ensure that you have the `University of York VPN <https://www.york.ac.uk/it-services/tools/vpn/>`_ set up. 

Logging in
------------

On MacOS and Linux, open the Terminal app

On Windows, download `MobaXTerm <https://mobaxterm.mobatek.net>`_ or a terminal emulator of your choice.

To log in to Viking, type: 

.. code-block:: console

   ssh abc123@viking.york.ac.uk

It will prompt you to type your password. 

.. note::

   Useful tip: for Mac and Linux, you can create an alias for logging in by updating the .bashrc file on your personal computer to include the following: 

.. code-block:: console

   alias viking='ssh -Y abc123@viking.york.ac.uk' (possibly -X)

If you need to log into a specific node (e.g. to check a screen), use the following: 

.. code-block:: console

   ssh abc123@viking-login1.york.ac.uk

   ssh abc123@viking-login2.york.ac.uk

If you don’t want to enter a password every time you log in to Viking:

1. On your local machine 

.. code-block:: console   

   cd ~/.ssh

2. Create a public/private key to create your identity file

Type 

.. code-block:: console

   ssh-keygen 

Generating public/private rsa key pair.

Enter file in which to save the key

.. code-block:: console   

   (/Users/username/.ssh/id_rsa):
 
Enter ``viking.key`` as the name 
 
Then it will ask
``Enter passphrase (empty for no passphrase):``
 
Just press enter to avoid typing a password every single time you log in to the server (it will ask twice). 
 
Then it will generate the key’s randomart image (visual representation of your key).
 
3. In the ssh directory, create a config file using nano (or a text editor of your choice)

.. code-block:: console 

   nano config

In the config file, copy paste the following (replace ``abc123`` with your username):

.. code-block:: console 

   Host viking
    Hostname 10.0.13.21
    User abc123
    IdentityFile ~/.ssh/viking.key
 
4. Then deploy the private key to the machine 

.. code-block:: console 

   ssh-copy-id -i ~/.ssh/viking.key viking
 
5. Now type 

.. code-block:: console 

   ssh viking 

It may ask for your password the first time around, but should not ask after.

