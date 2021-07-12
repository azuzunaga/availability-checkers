require 'dotenv/load'
require 'bundler'
require 'json'
require 'mail'
require 'net/http'

Mail.defaults do
  delivery_method :smtp,
    address:              ENV['MAILGUN_SMTP_SERVER'] || "smtp.mailgun.org",
    port:                 ENV['MAILGUN_SMTP_PORT'] || 587,
    user_name:            ENV['MAILGUN_SMTP_LOGIN'],
    password:             ENV['MAILGUN_SMTP_PASSWORD'],
    authentication:       'plain',
    enable_starttls_auto: true
end

class Everlane
  PRODUCT_ID = 7290; # mens-court-sneaker-off-white-fog
  ITEM_ID = 52076; # F12M10

  def initialize
    @data = check_availability
  end

  def check_availability
    res = Net::HTTP.get_response(URI("https://www.everlane.com/api/v2/product_groups?product_permalink=mens-court-sneaker-off-white-fog"))

    available = JSON.parse(res.body)["products"]
      .filter { |e| e["id"] == 7290 }
      &.[](0)
      &.[]("variants")
      &.filter { |e| e["id"] == 52076 }
      &.[](0)
      &.[]("available")

    available && available > 0
  end
end

task :email do
  data = Everlane.new

  if data
    mail = Mail.new do
      from    "Everlane Checker <availabilitychecker@#{ENV['MAILGUN_DOMAIN']}>"
      to      ENV['TO']
      subject "#{Time.now.strftime("%b %d")}: Sneakers available!}"

      html_part do
        content_type 'text/html; charset=UTF-8'
        body '<a href="https://www.everlane.com/products/mens-court-sneaker-off-white-fog?collection=mens-tread-sneakers">Get them!</a>'
      end
    end

    mail.deliver
  end
end

task :test do
  file = 'highlights.log'
  File.open(file, "a") do |fp|
    fp.puts "Rake task test ran at: #{Time.now.inspect}\n"
  end
end

task default: [:email]
