require 'rake'
require 'erb'

desc "install the dot files into user's home directory"
task :install do
  install_homebrew if RUBY_PLATFORM.downcase.include?("darwin") 
  install_oh_my_zsh
  install_bins
end

task default: [:install] do
  switch_to_zsh
  replace_all = false
  files = Dir['*'] - %w[Rakefile README.rdoc LICENSE oh-my-zsh retired]
  files.each do |file|
    system %Q{mkdir -p "$HOME/.#{File.dirname(file)}"} if file =~ /\//
    if File.exist?(File.join(ENV['HOME'], ".#{file.sub(/\.erb$/, '')}")) || true
      if File.identical? file, File.join(ENV['HOME'], ".#{file.sub(/\.erb$/, '')}")
        puts "identical ~/.#{file.sub(/\.erb$/, '')}"
      elsif replace_all
        replace_file(file)
      else
        print "overwrite ~/.#{file.sub(/\.erb$/, '')}? [ynaq] "
        case $stdin.gets.chomp
        when 'a'
          replace_all = true
          replace_file(file)
        when 'y'
          replace_file(file)
        when 'q'
          exit
        else
          puts "skipping ~/.#{file.sub(/\.erb$/, '')}"
        end
      end
    else
      link_file(file)
    end
  end
end

def replace_file(file)
  system %Q{rm -rf "$HOME/.#{file.sub(/\.erb$/, '')}"}
  link_file(file)
end

def link_file(file)
  if file =~ /.erb$/
    puts "generating ~/.#{file.sub(/\.erb$/, '')}"
    File.open(File.join(ENV['HOME'], ".#{file.sub(/\.erb$/, '')}"), 'w') do |new_file|
      new_file.write ERB.new(File.read(file)).result(binding)
    end
  elsif file =~ /zshrc$/ # copy zshrc instead of link
    puts "copying ~/.#{file}"
    system %Q{cp "$PWD/#{file}" "$HOME/.#{file}"}
  else
    puts "linking ~/.#{file}"
    system %Q{ln -s "$PWD/#{file}" "$HOME/.#{file}"}
  end
end

def switch_to_zsh
  if ENV["SHELL"] =~ /zsh/
    puts "using zsh"
  else
    print "switch to zsh? (recommended) [ynq] "
    case $stdin.gets.chomp
    when 'y'
      puts "switching to zsh"
      system %Q{chsh -s `which zsh`}
    when 'q'
      exit
    else
      puts "skipping zsh"
    end
  end
end

def install_oh_my_zsh
  if File.exist?(File.join(ENV['HOME'], ".oh-my-zsh"))
    puts "found ~/.oh-my-zsh"
  else
    print "install oh-my-zsh? [ynq] "
    case $stdin.gets.chomp
    when 'y'
      puts "installing oh-my-zsh"
      system %Q{git clone https://github.com/robbyrussell/oh-my-zsh.git "$HOME/.oh-my-zsh"}
    when 'q'
      exit
    else
      puts "skipping oh-my-zsh, you will need to change ~/.zshrc"
    end
  end
  puts "Copying custom themes"
  system %Q{cp -r themes/* "$HOME/.oh-my-zsh/themes"}
end

def run(cmd)
  puts "[Running] #{cmd}"
  `#{cmd}` unless ENV['DEBUG']
end

#Taken from skwp/dotfiles
def install_homebrew
  puts "======================================================"
  puts "Installing Homebrew, the OSX package manager...If it's"
  puts "already installed, this will do nothing."
  puts "======================================================"
  run %{ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"}
  puts
  puts
  puts "======================================================"
  puts "Installing Homebrew packages...There may be some warnings."
  puts "======================================================"
  run %{brew install ack}
  puts
  puts
end

# install other binaries
def install_bins
  run %{wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash}
  run %{brew install bat}
end
