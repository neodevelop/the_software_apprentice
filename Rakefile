namespace :build do

  desc "Build the HTML version"
  task :html do
    system("asciidoctor -o build/index.html master.adoc")
  end

  desc "Build the PDF version"
  task :pdf do
    system("asciidoctor-pdf -o build/book.pdf master.adoc")
  end

  desc "Build the ePub version"
  task :epub do
    system("asciidoctor-epub -D build master.adoc")
  end
end
