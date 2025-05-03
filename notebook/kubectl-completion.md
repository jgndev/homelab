# Enabling completion for kubectl

## Bash


## Zsh

### macOS

Install completion with Homebrew.

```
brew update
brew install zsh-completion
```

Update `.zshrc`

```
echo "alias k='kubectl'" >> ~/.zshrc

echo 'complete -o default -F __start_kubectl k' >> ~/.zshrc
```
