If you are trying to do the VSCode Overview Lab on your Mac, please follow the instructions below.

We need to set a NPM prefix, before actually installing a NPM package::

  mkdir $HOME/npm   #  this just creates a directory named _npm_ under $HOME, that's completely arbitrary, could be named anything
  npm config get prefix   #  if you see this they probably have something like /usr/local which a non-root userid can't update
  npm config set prefix $HOME/npm   #  now they can update this one
  npm config get prefix   #  to see that your command "took"

Below we will install a required NPM package, called ``node-gyp``::

  npm install -g node-gyp
  
Additionally, we need install the xcode command line tools::

  xcode-select --install

After these series of commands, you will be able to go through the lab found in my team's `Immersion Workshop <https://ibm-blockchain-wsc.github.io/ImmersionWorkshop/vscode-home/>`_
