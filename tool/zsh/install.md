### install zsh
```bash
# install zsh
sudo apt install zsh
# install oh my zsh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
# change default shell(also can edit /etc/passwd) 
chsh -s /bin/zsh
```
### install plugin
#### zsh-autosuggestions
add auto command completion(令提示)
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# add to .zshrc
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```