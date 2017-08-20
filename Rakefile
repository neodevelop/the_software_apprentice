require 'rest-client'
require 'json'
require 'base64'
require 'aws-sdk'

@token = ENV["POSTMARK_TOKEN"]
@from_recipient = ENV["FROM_RECIPIENT"]
@mail_recipients = ENV["MAIL_RECIPIENTS"]
@template_id = ENV["TEMPLATE_ID"]

# @access_key = ENV["AWS_ACCESS_KEY_ID"]
# @secret_key = ENV["AWS_SECRET_ACCESS_KEY"]
# @region = ENV["AWS_REGION"]
@bucket_name = ENV["AWS_BUCKET_NAME"]

task :default => 'notification:send'

namespace :build do

  desc "Build the HTML version"
  task :html do
    system("asciidoctor -o build/index.html master.adoc")
    puts "HTML Generated in build/"
  end

  desc "Build the PDF version"
  task :pdf do
    system("asciidoctor-pdf -o build/book.pdf -a pdf-stylesdir=themes -a pdf-style=basic master.adoc")
    puts "PDF Generated in build/"
  end

  desc "Build the ePub version"
  task :epub do
    system("asciidoctor-epub3 -D build -o book.epub master.adoc")
    puts "ePub Generated in build/"
  end

end

namespace :publish do
  desc "List all buckets"
  task :upload do
    s3 = Aws::S3::Resource.new
    bucket = s3.bucket(@bucket_name)
    files = Dir.glob('build/*').select { |e| File.file? e }
    files.each do |f|
      file = File.new f
      f.slice! "build/"
      s3_object = bucket.object(f)
      s3_object.upload_file file
      puts "#{f} uploaded..."
    end
  end
end

namespace :dictation do

  desc "Makes the correct capitalization"
  task :capitalize, [:filename] do |t, args|
    s = File.read(args[:filename]).to_s
    puts s.downcase.capitalize.gsub(/[a-z][^.?!]*/) { |match| match[0].upcase + match[1..-1].rstrip }
  end
end

namespace :notification do

  desc "Create the changelog with 5 last commits"
  task :changelog do
    sh "git log --oneline  -n 5 > log_history"
  end

  desc "Send an email with postmark"
  task :send => [:changelog, "build:pdf"] do
    headers = get_headers_for_postmark
    payload = create_payload
    url = "https://api.postmarkapp.com/email/withTemplate"
    begin
      response = RestClient.post(url, payload.to_json, headers)
      parsed_response = JSON.parse(response)
      puts parsed_response
    rescue  RestClient::ExceptionWithResponse => e
      puts e.response.to_s
    end
  end

end

private

def get_headers_for_postmark
  {
    "Accept" => "application/json",
    "X-Postmark-Server-Token" => @token
  }
end

def create_payload
  {
    "From" => @from_recipient,
    "To" => @mail_recipients,
    "TemplateId"  => @template_id,
    "TemplateModel" => {
      "changelog" => File.read("log_history").to_s.split("\n").map { |s| {message:s} }
    },
    "Attachments" => [
      {
        "Name" => "book.pdf",
        "ContentType" => "application/octet-stream",
        "Content" => Base64.encode64(File.read("build/book.pdf"))
      }
    ]
  }
end
