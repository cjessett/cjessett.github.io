---
layout: post
title: "Create Github repos from the command line"
date: 2016-10-01T03:55:31-05:00
---

### Because why not

This is an annotated version of [this original post](https://www.viget.com/articles/create-a-github-repo-from-the-command-line) and a useful intro into bash scripting.

The command that does the work:

```

curl -u "$username:$token" https://api.github.com/user/repos -d '{"name":"'$repo_name'"}' > /dev/null 2>&1

```

Where `$username`, `$token`, and `$repo_name` are your github username, personal access token, and desired repository name respectively.

We can wrap this in a sweet bash function to do this on the fly from a local git repo. In your `~/.bash_profile`:

```

github-create() {

  # assign $repo_name to argument if present
  repo_name=$1

  # capture the name of the current directory
  dir_name=`basename $(pwd)`

  # assign user input to $repo_name if argument was empty
  if [ "$repo_name" = "" ]; then
  echo "Repo name (hit enter to use '$dir_name')?"
  read repo_name
  fi

  # assign the current directory to $repo_name if not specified by input above
  if [ "$repo_name" = "" ]; then
  repo_name=$dir_name
  fi

}

```

This creates a function you can call in your terminal and pass an optional argument. The first line will store the argument passed to the function in the variable `$repo_name`.

The next line creates a default repository name in the case no argument is passed to the function. The command `basename $(pwd)` will print the name of the current directory, try it in the terminal.

Next, a conditional to check for the presence of an argument. If no argument was provided, the conditional reads from user input and assigns the result to `$repo_name`.

The last conditional assigns the current directory name to `$repo_name` in the case no input was provided. If you're curious about what's going on here add `echo "$repo_name"` at the bottom of the function then open a new terminal tab and call the function with  `github-create`. Then try it with an argument.

Next, we'll set the username and token variables to use in our API call.

```

github-create() {

  repo_name=$1

  dir_name=`basename $(pwd)`

  if [ "$repo_name" = "" ]; then
  echo "Repo name (hit enter to use '$dir_name')?"
  read repo_name
  fi

  if [ "$repo_name" = "" ]; then
  repo_name=$dir_name
  fi

  # this sets $username to the result of the command 'git config github.user'
  username=`git config github.user`
  if [ "$username" = "" ]; then
  echo "Could not find username, run 'git config --global github.user <username>'"
  invalid_credentials=1
  fi

  # this sets $token to the result of the command 'git config github.token'
  token=`git config github.token`
  if [ "$token" = "" ]; then
  echo "Could not find token, run 'git config --global github.token <token>'"
  invalid_credentials=1
  fi

  # return from the function if the credentials are not found
  if [ "$invalid_credentials" == "1" ]; then
  return 1
  fi

}

```

Here we check for a github username and token that have been set in the `~/.gitconfig` file. You can find info about setting options with `git config` [here](https://git-scm.com/docs/git-config).

To check if you have set git config options run `git config -l` to list all of your config options. You can also run ` git config ` for help with the command.

Running the command `git config --global github.user <YOUR_USERNAME>`, and likewise with `github.token`, will write to `~/.gitconfig` like so:

```
# ~/.gitconfig

[github]
  user: <YOUR_USERNAME>
  token: <YOUR_TOKEN>

```


I set my username and token with the commands above after [generating](https://github.com/settings/tokens) a [personal API token](//github.com/blog/1509-personal-api-tokens) with appropriate scopes. This allows you to make authenticated requests to your [GitHub](//github.com) account.

Now lets add the curl request.

```

github-create() {

  repo_name=$1

  dir_name=`basename $(pwd)`

  if [ "$repo_name" = "" ]; then
  echo "Repo name (hit enter to use '$dir_name')?"
  read repo_name
  fi

  if [ "$repo_name" = "" ]; then
  repo_name=$dir_name
  fi

  username=`git config github.user`
  if [ "$username" = "" ]; then
  echo "Could not find username, run 'git config --global github.user <username>'"
  invalid_credentials=1
  fi

  token=`git config github.token`
  if [ "$token" = "" ]; then
  echo "Could not find token, run 'git config --global github.token <token>'"
  invalid_credentials=1
  fi

  if [ "$invalid_credentials" == "1" ]; then
  return 1
  fi

  # echo some useful feedback and make a request to the API
  echo -n "Creating Github repository '$repo_name' ..."
  curl -u "$username:$token" https://api.github.com/user/repos -d '{"name":"'$repo_name'"}' > /dev/null 2>&1
  echo " done."

}

```

This makes a request to the `/user/repos` [endpoint](https://developer.github.com/v3/repos/#create) using [basic authentication](https://developer.github.com/v3/auth/#basic-authentication) to create a new GitHub repository.

That last bit of the `curl` command, `> /dev/null 2>&1` basically redirects and discards the standard output of the request, check out [this stackoverflow question](https://stackoverflow.com/questions/10508843/what-is-dev-null-21#10508862) for more info.

Finally, we add a remote named `origin` for the repo we just created and push up our code.

```

github-create() {

  repo_name=$1

  dir_name=`basename $(pwd)`

  if [ "$repo_name" = "" ]; then
  echo "Repo name (hit enter to use '$dir_name')?"
  read repo_name
  fi

  if [ "$repo_name" = "" ]; then
  repo_name=$dir_name
  fi

  username=`git config github.user`
  if [ "$username" = "" ]; then
  echo "Could not find username, run 'git config --global github.user <username>'"
  invalid_credentials=1
  fi

  token=`git config github.token`
  if [ "$token" = "" ]; then
  echo "Could not find token, run 'git config --global github.token <token>'"
  invalid_credentials=1
  fi

  if [ "$invalid_credentials" == "1" ]; then
  return 1
  fi

  echo -n "Creating Github repository '$repo_name' ..."
  curl -u "$username:$token" https://api.github.com/user/repos -d '{"name":"'$repo_name'"}' > /dev/null 2>&1
  echo " done."

  # add repo to remote as 'origin' and push the code up
  echo -n "Pushing local code to remote ..."
  git remote add origin git@github.com:$username/$repo_name.git > /dev/null 2>&1
  git push -u origin master > /dev/null 2>&1
  echo " done."

}

```

Now if you open a new terminal window you can call `github-create` with an optional argument and you won't have to leave the comfort of that terminal to create a new GitHub repo!
