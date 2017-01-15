require 'html-proofer'

opts = {
  :assume_extension => true,
  :file_ignore => [/\/assets.*/,/\/slides.*/]
}

task :default do
  sh "bundle exec jekyll build"
  HTMLProofer.check_directory("./_site", opts).run
end

# commandline:
# bundle exec htmlproofer ./_site --assume-extension --file-ignore /\/assets.*/,/\/slides.*/
