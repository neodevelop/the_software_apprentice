task :default => 'build:html'

namespace :build do

  desc "Build the HTML version"
  task :html do
    system("asciidoctor -o build/index.html master.adoc")
    puts "HTML Generated in build/"
  end

  desc "Build the PDF version"
  task :pdf do
    system("asciidoctor-pdf -o build/book.pdf master.adoc")
    puts "PDF Generated in build/"
  end

  desc "Build the ePub version"
  task :epub do
    system("asciidoctor-epub -D build master.adoc")
    puts "ePub Generated in build/"
  end
end
